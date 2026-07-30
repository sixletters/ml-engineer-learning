---
layout: page
title: ML Engineering & MLOps
permalink: /ml-engineering/
---

The gap between data scientist and ML engineer. Production ML systems need more than good models — they need tracking, reproducibility, serving, and monitoring.

## Topics

| Topic | Depth |
|-------|-------|
| MLflow: tracking, model registry, serving, projects | Deep — know all four components |
| Feature stores: Feast architecture, point-in-time retrieval, online/offline split | Deep |
| Model serving: batch scoring, real-time API, streaming | Deep |
| Train/serve skew — causes, detection, prevention | Deep |
| Model versioning and rollback | Deep |
| Model monitoring: data drift, concept drift, performance degradation | Deep |
| CI/CD for ML: training pipelines as code, automated retraining triggers | Medium |
| Containerisation: Docker, Docker Compose, building lean images | Deep |
| REST API design for ML: async endpoints, batching, timeouts | Deep |

## Resources

- **Designing Machine Learning Systems — Chip Huyen** — the best book on production ML, read this first
- **Machine Learning Engineering — Andriy Burkov** — practical, concise
- [Made With ML (madewithml.com)](https://madewithml.com) — free MLOps course, code-heavy
- [MLflow documentation](https://mlflow.org/docs/latest/index.html) — work through all tutorials
- [Feast documentation](https://docs.feast.dev) — architecture deep-dive section
- [Evidently documentation](https://docs.evidentlyai.com) — monitoring patterns

## Key Exercises

- Log a full training run in MLflow: hyperparams, dataset hash, git commit SHA, model artifact. Load the model back from the registry and run inference — if you can't reproduce from logged metadata alone, it's not logged correctly
- Reproduce train/serve skew deliberately — train with one feature encoding, serve with a slightly different one, observe degradation. Fix by packaging preprocessing with the model using `PythonModel`
- Build a shadow deployment — run two model versions in parallel, log both predictions without serving the challenger, compare distributions with Evidently
- Write a retraining trigger: query prediction logs, compute PSI on a key feature (implement the formula, no library), warn if drift exceeds a threshold
- Containerise a model: FastAPI + Dockerfile, build and run locally, call with `curl`
- Design a feature store schema for a real domain — batch vs on-demand features, entity keys, TTL. Implement in Feast

---

*Notes and exercises will be added below as I work through this section.*
