# Word Counter CLI — Implementation Plan

> **Single source of truth for autonomous execution.**

## Assumptions
- Python 3.10+ available
- No external dependencies beyond stdlib
- Cross-platform (Windows, macOS, Linux)

## Implementation Checklist

- [ ] Create `wc/` package directory with `__init__.py`
- [ ] Create `wc/cli.py` with argparse entry point accepting: `--file` (path), `--words` (flag), `--lines` (flag), `--chars` (flag), `--bytes` (flag)
- [ ] Implement `count_words(text)` in `wc/counter.py` — returns word count as int
- [ ] Implement `count_lines(text)` in `wc/counter.py` — returns line count
- [ ] Implement `count_chars(text)` in `wc/counter.py` — returns character count (excluding whitespace option)
- [ ] Implement `count_bytes(file_path)` in `wc/counter.py` — returns file size in bytes
- [ ] Wire CLI to counter functions: read file, call requested counters, print results
- [ ] Default behavior (no flags): output lines, words, bytes (like Unix wc)
- [ ] Handle file-not-found with clear error message and exit code 1
- [ ] Handle empty file: output zeros, not errors
- [ ] Handle binary files: detect and warn, still show byte count
- [ ] Create `pyproject.toml` with `[project.scripts]` entry point `wc = wc.cli:main`
- [ ] Write unit test `tests/test_counter.py` covering: normal text, empty string, multiline, unicode, single word, all spaces
- [ ] Write integration test `tests/test_cli.py` covering: all flags, no flags, missing file, empty file, binary file
- [ ] Verify all tests pass with `python -m pytest tests/ -v`
- [ ] Create `README.md` with install instructions, usage examples, and flag reference
