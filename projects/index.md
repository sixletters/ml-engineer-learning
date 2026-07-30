---
layout: page
title: Projects
permalink: /projects/
---

End-to-end portfolio projects ordered by complexity. Each builds on the last. Structure every project as a proper Python package — not a notebook dump.

```
my-project/
├── src/myproject/
├── tests/
├── Makefile
├── pyproject.toml
└── README.md   ← explain design decisions, not just what it does
```

---

## Tier 1 — Foundation

### Project 0: ML Math Visualization Platform

Implement ML mathematical foundations from scratch and visualise them interactively. Build alongside Month 1-2.

**Tech stack:** Python, NumPy, Matplotlib, Plotly, Streamlit

**What to build:** Gradient descent visualiser → matrix transformation visualiser → neural network playground → PCA/SVD visualiser → backprop step visualiser

See [Visualization](../visualization/) for the full spec.

**Status:** Not started

---

### Project 1: End-to-end Recommender System

Touch every layer of a real recommendation pipeline.

**Dataset:** [MovieLens 25M](https://grouplens.org/datasets/movielens/25m/)

**What to build:**
1. Ingest raw interaction data, clean and store as Parquet/Delta
2. Train ALS collaborative filtering using PySpark MLlib
3. Build a FAISS index over item embeddings for ANN search
4. Serve recommendations via a FastAPI endpoint
5. Track all experiments with MLflow

**Stretch:** Replace ALS with a two-tower neural retrieval model in PyTorch

**Key skills:** PySpark MLlib, FAISS, FastAPI, MLflow, Delta Lake

**Status:** Not started

---

### Project 2: Feature Pipeline with a Feature Store

Building this properly signals seniority — most ML engineers skip it.

**What to build:**
1. Pick a dataset with temporal structure (user purchase history or click logs)
2. Engineer point-in-time correct features using Spark (no leakage across the time axis)
3. Register and serve features via Feast (open source, self-hosted)
4. Train a gradient boosting model (XGBoost or LightGBM) consuming features from the store
5. Log the full training run in MLflow — feature versions, dataset snapshot, metrics

**Why it matters:** Feature stores enforce reproducibility and prevent train/serve skew — the #1 silent killer of production ML.

**Key skills:** Feast, PySpark, point-in-time joins, XGBoost/LightGBM, MLflow

**Status:** Not started

---

## Tier 2 — Differentiation

### Project 3: Real-time Ranking System

Batch recommendations are table stakes. Real-time is where hard engineering lives.

**Architecture:**
```
User event → Kafka → Bytewax (feature computation) → Redis (feature cache)
                                                           ↓
Request → FastAPI → Qdrant (retrieval) → re-ranker model → response
```

**What to build:**
1. Simulate user event stream using Kafka (Redpanda locally — same API, easier setup)
2. Compute rolling/windowed features in real-time using Bytewax
3. At request time: retrieve candidates from Qdrant, re-rank with a lightweight model
4. Serve via FastAPI with sub-100ms p99 latency as the target

**Key skills:** Kafka/Redpanda, Bytewax, Qdrant, Redis, latency profiling

**Status:** Not started

---

### Project 4: LLM-Powered Automation Agent

Combine automations work with LLM engineering — a highly employable combination.

**What to build:**
1. Pick a real tedious workflow (expense categorisation, document parsing, report generation)
2. Build a multi-step agent using LangGraph with tools, memory, and conditional branching
3. Use Pydantic v2 for all structured inputs/outputs — no raw string parsing
4. Log all traces and LLM calls with Langfuse (open source, self-hosted)
5. Expose as a REST API (FastAPI) or Slack bot

**Key skills:** LangGraph, Anthropic SDK, Pydantic v2, Langfuse, FastAPI

**Status:** Not started

---

### Project 5: A/B Testing Framework from Scratch

Understanding how ML decisions get validated in production. Almost nobody builds this themselves — it stands out.

**What to build:**
1. Implement a Bayesian A/B test engine in pure Python (no statsmodels shortcuts)
2. Simulate a recommendation system with two variants
3. Track conversion metrics per variant, compute credible intervals, implement early stopping logic
4. Build a Streamlit dashboard showing live experiment results and decision thresholds

**Key skills:** Bayesian statistics, pure Python numerical code, Streamlit, experiment design

**Status:** Not started

---

## Tier 3 — Senior-level Signal

### Project 6: Mini ML Platform

Shows systems thinking and infrastructure understanding, not just modelling ability.

**What to build:**
1. Feature computation job (PySpark) → writes to Delta table
2. Model training job (MLflow) → reads features, trains, registers model
3. FastAPI serving layer → loads model from registry, handles requests
4. Redis cache → low-latency feature lookups at serve time
5. Model monitoring → log predictions, detect distribution drift with Evidently
6. Containerise everything with Docker Compose — one command to bring it all up

**Key skills:** Docker Compose, MLflow model registry, Evidently, Redis, end-to-end system design

**Status:** Not started

---

### Project 7: Open Source Contribution

Real code review from experienced engineers. Visible on GitHub permanently.

**Good targets:**
- [MLflow](https://github.com/mlflow/mlflow) — experiment tracking, model registry
- [Feast](https://github.com/feast-dev/feast) — feature store
- [Evidently](https://github.com/evidentlyai/evidently) — ML monitoring
- [Bytewax](https://github.com/bytewax/bytewax) — stream processing in Python
- [LightFM](https://github.com/lyst/lightfm) — hybrid recommendation library

**How to start:** Fix a docs issue or add test coverage first to understand the codebase. Then pick a real `good first issue` bug.

**Status:** Not started

---
