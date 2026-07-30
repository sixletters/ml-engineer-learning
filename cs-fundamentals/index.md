---
layout: page
title: CS Fundamentals
permalink: /cs-fundamentals/
---

Algorithms, data structures, SQL, networking, and software engineering practices. These underpin everything in the roadmap — if they're shaky, the advanced material won't stick.

## Algorithms & Data Structures

| Topic | Why It Matters |
|-------|---------------|
| Big-O notation, time/space complexity | Understand why ANN search is O(log n) vs brute force O(n) |
| Hash maps, trees, heaps, graphs | Used everywhere in pipeline and serving code |
| Sorting and search algorithms | Interview staple; informs understanding of indexing |

## Networking & SQL

| Topic | Why It Matters |
|-------|---------------|
| HTTP, REST, TCP/IP, latency vs throughput | Needed for serving, API design, debugging production issues |
| SQL: joins, aggregations, window functions, indexes, query plans | Feature engineering, debugging pipelines |

## Software Engineering Practices

| Topic | Why It Matters |
|-------|---------------|
| Git: branching, rebasing, conflict resolution, bisect | Daily use; bisect is a superpower for finding regressions |
| Clean code: naming, single responsibility, avoiding premature abstraction | Production ML code is read far more than it's written |
| Testing: unit vs integration vs e2e | Confidence in changes, catching regressions |
| Shell: pipes, grep, awk, sed, find, xargs | Debugging production jobs, data wrangling |

## Resources

- **Grokking Algorithms — Aditya Bhargava** — visual and accessible
- **LeetCode Easy/Medium** — 30-50 problems focused on arrays, hashmaps, trees, sliding window
- **Mode Analytics SQL Tutorial** (free); **SQLZoo** for practice
- **Pro Git — Scott Chacon** (free online)
- **A Philosophy of Software Design — John Ousterhout**
- **Python Testing with pytest — Brian Okken**
- **The Linux Command Line — William Shotts** (free PDF)

## Key Exercises

- LeetCode: 2 Easy + 1 Medium per session for 4-6 weeks — arrays, hashmaps, sliding window patterns
- Write raw SQL for 10 business questions on a dataset you care about, window functions for at least 3
- `EXPLAIN ANALYZE` a slow SQL query in local Postgres, rewrite it to use an index
- Use `git bisect` to find which commit broke something — introduce a bug yourself, then find it
- Shell one-liner: find `.log` files modified in the last 24h, extract ERROR lines, count by type, sort descending

---

*Notes and exercises will be added below as I work through this section.*
