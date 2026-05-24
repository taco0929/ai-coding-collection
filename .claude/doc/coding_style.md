# Coding Style

## Imports

Place all imports at the top of the module. Do not import inside functions or methods unless the import would cause a circular dependency.

## Data Carriers

Do not use `dict` as a data carrier between functions or across module boundaries. Define a Pydantic model instead. This applies to function arguments, return values, and internal data passed between components.

## Type Hints

Use `typing` module types for all type hints — not built-in generic classes.

```python
# correct
from typing import List, Optional, Dict
def get_items() -> List[str]: ...

# wrong
def get_items() -> list[str]: ...
```

## File Formatting

Leave a single blank line at the end of every file.

## Function Design

Keep functions focused and short. A function should do one thing. Avoid:

- Functions that span many decision branches — extract each branch into a named helper
- Long functions that mix orchestration with business logic — separate the two
- Functions with many parameters that signal unclear responsibility — introduce a model or split the function

Prefer shallow call trees of well-named, single-purpose functions over deep, complex ones.
