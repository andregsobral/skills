---
name: write-prd
description: Write a Product Requirement Document in Markdown for the task currently being discussed in the session.
user-invocable: true
argument-hint: [optional short title or one-liner override]
---

# Write PRD Skill

Synthesise the current conversation into a concise, well-structured PRD in Markdown and print it directly in the chat. Do not save a file unless the user explicitly asks.

---

## When to Use

Invoke with `/write-prd` at any point in a session where a feature, tool, or product change is being discussed. The skill reads the conversation context to understand what is being built and for whom.

If the user passes an argument (e.g. `/write-prd "OAuth login"`), treat it as the definitive title/scope and use the session context only to fill in supporting detail.

---

## Step-by-Step Workflow

### Step 1 — Understand the Task

Read the current session context carefully. Identify:

- **What** is being built or changed
- **Why** — the problem or motivation driving it
- **Who** — the user(s) or system(s) affected
- **Constraints** — any technical, time, or scope limits already mentioned

If the conversation does not contain enough information to fill the core sections (Problem Statement, Goals, Requirements), ask **one focused question** before proceeding — not a list of questions. Example:

> "Before I write the PRD, could you clarify who the primary user of this feature is?"

If the task is clear enough to make reasonable inferences, proceed without asking.

### Step 2 — Draft the PRD

Produce a Markdown document using the structure below. Keep each section concise — a PRD is a communication tool, not an essay.

```markdown
# PRD: <Title>

## Overview
One or two sentences describing what this document covers.

## Problem Statement
What problem are we solving? Why does it matter now?
Focus on the pain, not the solution.

## Goals
- What success looks like — use measurable outcomes where possible.
- List 3–5 goals maximum.

## Non-Goals
- What is explicitly out of scope for this effort.
- Helps prevent scope creep.

## Users & Use Cases
Who is affected and how?

| User | Use Case |
|------|----------|
| ...  | ...      |

## Functional Requirements
Numbered list of things the system **must** do.

1. ...
2. ...

## Non-Functional Requirements
Performance, security, reliability, scalability constraints.

- ...

## Technical Considerations
Known constraints, dependencies, or architectural notes surfaced in the session.

## Success Metrics
How will we know this is working? Quantify where possible.

- ...

## Open Questions
Things that need a decision before or during implementation.

- [ ] ...
```

### Step 3 — Print the PRD

Output the completed PRD as a fenced Markdown code block so it can be copied cleanly:

````
```markdown
# PRD: ...
...
```
````

After printing, add a one-line note:

> _PRD based on this session's context — let me know if any section needs adjusting._

---

## Quality Bar

- **Be specific.** Vague goals like "improve performance" should be "p99 latency under 200 ms on the search endpoint."
- **Be brief.** Each section should be scannable. Prefer bullet points over prose.
- **Stay honest.** If the session doesn't provide enough detail to fill a section, write "TBD — needs input" rather than making things up.
- **No fluff.** Skip boilerplate phrases like "This document describes…" or "The purpose of this PRD is…".
