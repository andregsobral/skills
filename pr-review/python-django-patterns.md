# Python 3.13 & Django Code Review Patterns

Focus on these patterns when reviewing Python/Django code. Prefer flagging real bugs and meaningful improvements — avoid nitpicking style that's already consistent in the codebase.

---

## Python 3.13 Patterns

### Type Hints & Modern Syntax

- Use `X | Y` union syntax (Python 3.10+) instead of `Optional[X]` or `Union[X, Y]`
- Use `X | None` instead of `Optional[X]`
- Prefer `list[int]` over `List[int]`, `dict[str, Any]` over `Dict[str, Any]` (no import needed from 3.9+)
- Use `typing.TypedDict` or `dataclasses.dataclass` for structured dicts
- Flag missing type hints on public functions/methods (not private helpers)

### Control Flow

- Prefer `match` statement over long `if/elif` chains with literal comparisons
- Use walrus operator `:=` where it genuinely simplifies code (not just to be clever)
- Flag bare `except:` — should always catch specific exceptions

### Datetime

- `datetime.utcnow()` is deprecated in 3.12 — use `datetime.now(UTC)` or `datetime.now(timezone.utc)`
- Always use timezone-aware datetimes; flag naive datetimes in Django projects

### Resources & Context Managers

- Files, DB connections, locks must use `with` statement
- Flag any `f.open()` / `f.close()` pattern not wrapped in `with`

### Strings & Formatting

- Flag old-style `%s` formatting and `.format()` where f-strings are clearer
- Flag f-strings with complex expressions that should be extracted to variables

### Collections & Iteration

- Use generator expressions instead of list comprehensions when result is only iterated once
- Use `itertools` for complex iteration patterns
- Flag creating a full list just to check membership — use a `set` or `any()`

### Exception Handling

- Always use `raise NewError(...) from original_error` when re-raising in a different exception type
- Flag swallowed exceptions (`except Exception: pass`)
- Flag `except Exception as e: logger.error(e)` without re-raising — silently eats errors

### Mutable Defaults

- Flag mutable default arguments: `def f(items=[])`, `def f(config={})` — use `None` sentinel

### Pathlib

- Prefer `pathlib.Path` over `os.path` string concatenation for new code

### Dataclasses

- Flag `__init__` boilerplate that could be `@dataclass`
- Use `field(default_factory=...)` for mutable defaults in dataclasses

---

## Django Patterns

### Query Optimization (highest priority)

- **N+1 queries**: Flag loops that access related objects without `select_related` or `prefetch_related`
  ```python
  # Bad — N+1
  for order in Order.objects.all():
      print(order.user.email)  # hits DB each iteration

  # Good
  for order in Order.objects.select_related("user"):
      print(order.user.email)
  ```
- Flag `.all()` followed by Python-level filtering — use queryset filters instead
- Flag `len(queryset)` — use `.count()` for existence checks; use `queryset.exists()` for boolean checks
- Flag `queryset[0]` without `.first()` — raises IndexError on empty
- Flag missing `only()` or `defer()` when fetching wide tables but using few fields

### Atomic Operations

- Flag non-atomic "read-modify-write" patterns — use `F()` expressions:
  ```python
  # Bad
  obj.counter += 1
  obj.save()

  # Good
  MyModel.objects.filter(pk=obj.pk).update(counter=F("counter") + 1)
  ```
- Flag multiple DB writes that should be in `transaction.atomic()`
- Flag `save()` inside a loop — prefer `bulk_create` / `bulk_update`

### ORM Best Practices

- Flag raw SQL strings (`cursor.execute(...)`) unless there's a clear reason
- Flag `get()` without try/except `DoesNotExist` — or suggest `.filter().first()`
- Use `get_or_create`, `update_or_create` instead of manual check-then-create
- Use `Q()` for complex OR/AND filter logic
- Use `annotate()` + aggregations instead of post-processing in Python

### Security

- Flag `mark_safe()` on user-supplied content — XSS risk
- Flag direct string interpolation in `RawSQL`, `extra()`, `cursor.execute()` — SQL injection
- Flag `@csrf_exempt` without clear justification
- Flag serializers/forms that pass `request.data` directly without validation

### User Model

- Flag `from django.contrib.auth.models import User` direct import — use `get_user_model()` or `settings.AUTH_USER_MODEL`

### URLs

- Flag hardcoded URL strings — use `reverse()` or `reverse_lazy()`
- Flag `redirect("/some/path")` — should use `reverse()`

### Settings

- Flag secrets or environment-specific config hardcoded in settings files
- Flag `DEBUG = True` in production-facing settings

### Models

- Flag models missing `__str__`
- Flag models with no `Meta.ordering` that are displayed in lists (ordering should be explicit)
- Flag `null=True` on string fields (`CharField`, `TextField`) — Django convention is to use `blank=True` + empty string; `null=True` causes two possible empty values

### Signals

- Flag signal usage where a model method or service layer would be clearer
- Flag signals that trigger DB queries inside a `post_save` without `update_fields` guard

### Django REST Framework

- Flag serializer `create`/`update` that bypass `validated_data`
- Flag views that access `request.data` outside a serializer
- Flag missing `permission_classes` on API views
- Flag `SerializerMethodField` that triggers DB queries — N+1 risk

### Migrations

- Flag `RunPython` without a reverse migration function when data is modified
- Flag schema changes that could lock large tables (adding non-null columns without defaults)
