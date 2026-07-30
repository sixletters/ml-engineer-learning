---
layout: page
title: Data Engineering
permalink: /data-engineering/
---

PySpark, Delta Lake, and production-grade feature pipelines. Directly relevant to Databricks work — go deep.

## Topics

| Topic | Depth |
|-------|-------|
| Spark execution model: DAGs, stages, tasks, shuffle | Deep — needed to debug performance |
| Spark SQL: window functions, lateral joins, aggregate pushdown | Deep |
| Partitioning: repartition vs coalesce, how it affects performance | Deep |
| Delta Lake: ACID transactions, time travel, Z-ordering, OPTIMIZE | Deep |
| Broadcast joins vs sort-merge joins — when to use each | Deep |
| Catalyst optimiser: logical/physical plan, `explain()` output | Medium |
| Spark streaming: structured streaming, watermarks, triggers | Medium |
| Point-in-time correct feature engineering (no leakage) | Deep — critical for ML pipelines |
| Data skew: salting, adaptive query execution | Medium |

## Resources

- **Learning Spark, 2nd ed. — Damji et al.** — free PDF from Databricks, the standard text
- **Spark: The Definitive Guide — Chambers & Zaharia** — deeper reference
- [Databricks Academy](https://www.databricks.com/learn/training) — free courses, directly applicable
- [Delta Lake documentation](https://docs.delta.io/latest/index.html) — read the internals sections
- **High Performance Spark — Karau & Warren** — performance tuning focused

## Key Exercises

- Read and explain a query plan — run `.explain(True)` on a Spark job, write plain-English explanation of every stage. Spot the shuffle, broadcast join, and filter pushdown by eye
- Cause and fix a data skew — synthetic dataset with 90% records on one key, observe task time in Spark UI, fix with salting. Document before/after
- Window function drill: 10 different queries on one dataset (running totals, rank, lag/lead, first/last, percentiles) in both PySpark and Spark SQL
- Point-in-time feature join: user events table + slowly-changing feature table — join correctly so each event only uses the feature value valid at that moment
- OPTIMIZE and Z-ORDER experiment: measure scan metrics before/after on a Delta table
- Tune a slow shuffle-heavy job: use Spark UI to diagnose, apply two optimisations (broadcast hint, repartition, persist, AQE settings), measure improvement

---

*Notes and exercises will be added below as I work through this section.*
