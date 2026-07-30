---
layout: page
title: Python
permalink: /python/
---

Deep internals, ecosystem fluency, and advanced patterns. Python is the foundation — understanding how it actually works separates strong ML engineers from notebook users.

## Core Language Internals

| Topic | Depth |
|-------|-------|
| Memory model: reference counting, `gc`, weak refs, `tracemalloc` | Deep — understand when things get freed |
| The GIL: what it blocks, when it doesn't (multiprocessing vs asyncio) | Deep — critical for concurrency decisions |
| Generators and iterators: `yield`, `yield from`, lazy evaluation | Deep — used everywhere in data pipelines |
| Decorators: closures, `functools.wraps`, stacking, async decorators | Deep |
| Descriptors: `__get__`, `__set__`, `__delete__` — how `property` works | Medium |
| Metaclasses and `type` — understand it, don't overuse it | Medium |
| `asyncio`: event loop, coroutines, `gather`, `TaskGroup`, backpressure | Deep — needed for any async serving code |
| `dataclasses`, `__slots__`, `NamedTuple` | Deep — daily use |
| Import system: `sys.modules`, `importlib`, circular imports | Medium |
| Profiling: `cProfile`, `line_profiler`, `py-spy`, `memory_profiler` | Medium — know how to find bottlenecks |

## Ecosystem & Tooling

| Tool | Focus |
|------|-------|
| `uv` | Dependency resolution, lockfiles, virtualenvs, workspace mode |
| `pyproject.toml` | Full project config: build backend, tool config, metadata |
| `mypy` / `pyright` | Generics, `TypeVar`, `Protocol`, `Annotated`, strict mode |
| `pytest` | Fixtures (scope, autouse), parametrize, monkeypatch, conftest |
| `hypothesis` | Property-based testing — stateful testing is the hard part |
| `Pydantic v2` | Validators, computed fields, custom types, `model_config` |
| `ruff` | Linting + formatting — replaces flake8/black/isort |
| Pre-commit hooks | Enforce standards at commit time |

## Resources

- **Fluent Python (2nd ed.) — Luciano Ramalho** — the definitive deep-dive, read cover to cover
- **CPython Internals — Anthony Shaw** — walks through the actual CPython source
- **Python Cookbook, 3rd ed. — Beazley & Jones** — advanced recipes for descriptors/metaclasses
- [Python Data Model docs](https://docs.python.org/3/reference/datamodel.html) — read this fully once
- [uv docs](https://docs.astral.sh/uv/)
- [Pydantic v2 docs](https://docs.pydantic.dev/latest/)

## Key Exercises

- Build a lazy multi-stage generator pipeline: read large CSV line-by-line, filter, transform, aggregate without loading into memory. Profile memory before/after with `memory_profiler`
- Write a `@retry` decorator from scratch — configurable attempts, backoff, exception filtering. No libraries. Then make it work on async functions too
- Implement `@cached_property` using descriptors only — no `functools`. Test that it only computes once and is instance-scoped
- Build a connection pool with `asyncio.Queue` — fixed-size pool, async acquire/release with timeout
- Reproduce a memory leak (growing list inside a closure), detect and fix with `gc` and `tracemalloc`
- GIL experiment: CPU-bound task with threads vs processes, measure wall-clock time, explain the results

---

*Notes and exercises will be added below as I work through this section.*
