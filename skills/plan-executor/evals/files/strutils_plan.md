# String Utils Library — Implementation Plan

> **Single source of truth for autonomous execution.**

## Assumptions
- Python 3.10+
- Pure Python, no external dependencies
- Type hints throughout

## Implementation Checklist

- [ ] Create `strutils/` package directory with `__init__.py`
- [ ] Implement `slugify(text)` in `strutils/core.py` — converts to lowercase, replaces spaces with hyphens, removes non-alphanumeric chars except hyphens, collapses multiple hyphens, strips leading/trailing hyphens
- [ ] Implement `truncate(text, max_length, suffix="...")` in `strutils/core.py` — truncates to max_length minus len(suffix), appends suffix if truncated, never truncates mid-word if possible
- [ ] Implement `camel_to_snake(text)` in `strutils/core.py` — converts camelCase to snake_case
- [ ] Implement `snake_to_camel(text)` in `strutils/core.py` — converts snake_case to camelCase
- [ ] Implement `extract_emails(text)` in `strutils/core.py` — returns list of email addresses found in text using regex
- [ ] Implement `mask_string(text, visible_start=2, visible_end=2, mask_char="*")` — masks middle portion, handles strings shorter than visible_start + visible_end
- [ ] Add `__all__` to `__init__.py` exporting all public functions
- [ ] Create `pyproject.toml` with package metadata and `[project.optional-dependencies]` dev group with pytest
- [ ] Write tests in `tests/test_core.py`: 3+ cases per function including edge cases (empty string, None, already-hyphenated slug, string shorter than truncation limit, no camelCase, invalid email format)
- [ ] Verify all tests pass with `python -m pytest tests/ -v`
- [ ] Create `README.md` with function reference, examples, and install instructions
