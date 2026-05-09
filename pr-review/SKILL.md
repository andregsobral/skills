---
name: review-pr
description: Review a GitHub pull request for Python 3.13 / Django issues, present findings for selection, then post selected comments as a pending GitHub review (not submitted).
user-invocable: true
argument-hint: <PR-URL | PR-number> [repo]
---

# PR Code Review Skill

Review a GitHub pull request for Python 3.13 and Django issues, present numbered findings, and post selected ones as a **pending** GitHub review that the user submits manually.

---

## When to Use

Invoke with `/review-pr` followed by:
- A full GitHub PR URL: `/review-pr https://github.com/owner/repo/pull/42`
- A PR number (in the current repo): `/review-pr 42`
- A PR number and repo: `/review-pr 42 owner/repo`

---

## Step-by-Step Workflow

### Step 1 — Resolve the PR

Parse the argument to extract `owner`, `repo`, and `pull_number`.

If a full URL is provided, extract components from it.
If only a number is provided, infer the repo from:
```bash
gh repo view --json owner,name -q '"\(.owner.login)/\(.name)"'
```

Fetch PR metadata:
```bash
gh pr view <PR> --repo <owner/repo> --json number,title,url,baseRefName,headRefName,additions,deletions,changedFiles
```

### Step 2 — Fetch the Diff

```bash
gh pr diff <PR> --repo <owner/repo>
```

Parse the diff output carefully:
- Track current file from `diff --git a/... b/...` lines
- Track line numbers from `@@` hunk headers: `@@ -old_start,old_len +new_start,new_len @@`
- For each line in the diff:
  - Lines starting with `+` (added): `side = RIGHT`, line number increments from `new_start`
  - Lines starting with `-` (removed): `side = LEFT`, line number increments from `old_start`
  - Context lines (no prefix): increment both counters
- Keep a mapping of `(file, line_number, side)` for every reviewable line

Only review `.py` files. Skip `migrations/`, and any generated files unless a clear bug is present.

### Step 3 — Analyse the Code

Read and apply the rules in [python-django-patterns.md](./python-django-patterns.md).
Use the voice and phrase guide in [review-phrases.md](./review-phrases.md).

- List 5-7 potential issues or concerns for each file
- Gather evidence (check similar patterns, run tests, trace data flow)
- Narrow to 1-2 most critical issues per file
- Verify issues are real (not false positives or already handled)
- Only report confirmed, actionable feedback
- This ensures thorough but focused reviews without noise.

For each issue found, record:
- `file` — relative path (as it appears in the diff)
- `line` — the **new file** line number (integer) for the relevant line
- `side` — `RIGHT` for new/changed lines (almost always), `LEFT` only for removed lines
- `severity` — one of: `BUG`, `PERFORMANCE`, `SAFETY`, `STYLE`, `MINOR`
- `category` — short label, e.g. `N+1 query`, `mutable default`, `deprecated API`
- `comment` — the comment text, written in the voice from [review-phrases.md](./review-phrases.md)

**Quality bar:**
- Focus on `BUG`, `PERFORMANCE`, and `SAFETY` first
- Include `STYLE` and `MINOR` only if they're meaningful — not every f-string opportunity or import order
- Maximum ~15 findings total; if there are more, prefer the most impactful ones
- Do not comment on code that was not changed in this PR

Review the changes systematically, focusing on the areas below. Use the TodoWrite tool to track your review progress through different aspects.

### Correctness & Logic

- **Runtime errors**: Check for potential exceptions, null/undefined access, array out-of-bounds
- **Edge cases**: Empty arrays, null values, boundary conditions, concurrent access
- **Logic errors**: Off-by-one errors, incorrect conditionals, race conditions
- **Type safety**: Proper type annotations, avoiding `any` in TypeScript
- **Error handling**: Appropriate try-catch blocks, error propagation, user-friendly messages

### Performance

- **Algorithm complexity**: Avoid O(n²) or worse where O(n) or O(log n) is possible
- **Database queries**:
  - N+1 query problems (missing prefetch/select_related in Django)
  - Missing indexes for new query patterns
  - Unbounded queries without pagination
  - Inefficient joins or subqueries
- **Memory usage**: Unnecessary data copying, memory leaks, large object allocations
- **Caching**: Opportunities for caching expensive operations
- **Network calls**: Batching, unnecessary requests, missing timeouts

### Security

- **Injection vulnerabilities**: SQL injection, command injection, XSS, path traversal
- **Authentication & Authorization**: Proper permission checks, role validation
- **Data exposure**: Sensitive data in logs, error messages, or API responses
- **Input validation**: Sanitize and validate all user inputs
- **Secrets management**: No hardcoded credentials, API keys, or tokens
- **Dependency vulnerabilities**: Check for known CVEs in new dependencies
- **CORS & CSP**: Proper configuration for web applications

### Design & Architecture

- **Consistency**: Follows existing patterns and conventions in the codebase
- **Separation of concerns**: Clear boundaries between components/modules
- **DRY principle**: Avoid duplicating logic (but don't over-abstract)
- **SOLID principles**: Appropriate use of abstraction and interfaces
- **API design**: Clear contracts, versioning strategy, backward compatibility
- **Configuration**: Externalize environment-specific values
- **Error boundaries**: Proper error handling at system boundaries

### Code Quality

- **Readability**: Clear variable/function names, appropriate comments for complex logic
- **Formatting**: Follows project style guide (use linters/formatters)
- **Complexity**: Functions are focused and not too long
- **Documentation**: Public APIs have docstrings/JSDoc comments
- **Dead code**: Remove commented-out code, unused imports, unreachable code
- **Magic numbers**: Use named constants for unclear literal values

### Step 4 — Present Findings to User

Display findings as a numbered list. Group by severity (BUG > PERFORMANCE > SAFETY > STYLE > MINOR), then by file.

Format each finding as:

```
[N] [SEVERITY] file.py:line — Category
    "Proposed comment text"
```

Example:
```
[1] [BUG]  orders/views.py:87 — Unhandled DoesNotExist
    "This will raise `DoesNotExist` if the order isn't found — `.filter().first()` or
     a try/except block would handle it gracefully."

[2] [PERFORMANCE] orders/serializers.py:34 — N+1 query
    "This will hit the database once per order in the list. Adding
     `prefetch_related('items')` to the queryset in the view should fix it."

[3] [STYLE] orders/models.py:12 — Optional[X] → X | None
    "Minor: `Optional[str]` can be written as `str | None` in Python 3.10+."
```

After the list, ask:

> Which findings would you like to add to the pending review?
> Reply with numbers (e.g. `1 2 4`), `all`, `none`, or a range like `1-3`.
> You can also say `all except 3` or `bugs` (to add only BUG severity).

### Step 5 — Collect Selection

Parse the user's response into a list of selected finding indices. Handle:
- `all` → select everything
- `none` → exit, nothing to post
- `bugs` → select all `BUG` severity
- `performance` → select all `PERFORMANCE` severity
- `1 2 4` or `1,2,4` → select by number
- `1-5` → select range
- `all except 3 5` → exclude listed numbers

If any selected finding targets a line not in the diff (e.g. the line was removed, not added), skip it and warn the user.

### Step 6 — Post Pending Review

Build the comments array from selected findings:
```json
[
  {
    "path": "orders/views.py",
    "line": 87,
    "side": "RIGHT",
    "body": "This will raise `DoesNotExist` if the order isn't found..."
  }
]
```

Post the pending review (omitting `event` field leaves it in PENDING state):
```bash
gh api repos/{owner}/{repo}/pulls/{pull_number}/reviews \
  --method POST \
  --input - <<'EOF'
{
  "body": "",
  "comments": [ ... ]
}
EOF
```

**Important:** Do NOT include an `event` field. Omitting it creates a PENDING review that only you can see until you submit it manually on GitHub.

If the API call succeeds, confirm with:
```
Posted N comment(s) as a pending review on PR #{number}.

Go to: {PR URL}
Scroll to "Finish your review" at the bottom to review and submit.
```

If the API call fails (e.g. a line number is not part of the diff), report which comments failed and offer to retry or skip them.

---

## Edge Cases

- **Non-Python files only**: If the PR touches no `.py` files, say so and exit.
- **Very large diffs**: If the diff is over 2000 lines, note that only the first 2000 lines were analysed, and offer to continue in sections.
- **No issues found**: Report "No significant issues found in the changed Python code."
- **Draft PRs**: Still works — pending reviews on draft PRs are fine.

---

## Reference Files

- [python-django-patterns.md](./python-django-patterns.md) — What to look for
- [review-phrases.md](./review-phrases.md) — How to say it (add your own phrases to the USER PHRASES section)
