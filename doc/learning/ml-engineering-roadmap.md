# ML Engineering & Recommendations — Personal Development Roadmap

Focus areas: **Technical ML Engineering**, **Recommendation Systems**, **Python mastery and ecosystem fluency**

---

## Python Mastery — Learn This Deeply First

Before or alongside the projects, get fluent in how Python actually works — not just syntax.

### Core language internals

- The data model: `__dunder__` methods, descriptor protocol, `__slots__`
- How Python manages memory: reference counting, garbage collector, `gc` module
- The GIL — what it is, when it matters, when it doesn't (multiprocessing vs threading vs asyncio)
- Generators and iterators: `yield`, `yield from`, lazy evaluation patterns
- Decorators: closures, `functools.wraps`, class-based decorators, decorator factories
- Context managers: `__enter__`/`__exit__`, `contextlib.contextmanager`
- Metaclasses and class creation — understand how `type` works
- `asyncio` — event loop, coroutines, `async/await`, `gather`, `TaskGroup`

### Ecosystem fluency

| Area            | Tools                                                                  |
| --------------- | ---------------------------------------------------------------------- |
| Packaging       | `uv`, `pyproject.toml`, virtual envs, publishing to PyPI               |
| Type system     | `mypy`, `pyright`, generics, `TypeVar`, `Protocol`, `dataclasses`      |
| Testing         | `pytest`, fixtures, parametrize, `hypothesis` (property-based testing) |
| Profiling       | `cProfile`, `line_profiler`, `memory_profiler`, `py-spy`               |
| Concurrency     | `asyncio`, `concurrent.futures`, `multiprocessing`                     |
| Data validation | `Pydantic v2` — validators, computed fields, model serialisation       |
| CLI tooling     | `typer`, `rich` for terminal output                                    |

### Resources

- **Fluent Python (2nd ed.) — Luciano Ramalho** — the definitive deep-dive
- **CPython Internals — Anthony Shaw** — read the actual interpreter source
- **Python Cookbook (O'Reilly)** — advanced patterns

---

## Fundamentals — Do These Before Everything Else

These underpin everything in the roadmap. If they're shaky, the advanced material won't stick.

### Mathematics

| Topic                                                                              | Why it matters                                               | Resource                                                                                                             |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| Linear algebra: vectors, matrices, dot products, SVD, eigendecomposition           | Embeddings, matrix factorisation, PCA are all linear algebra | **3Blue1Brown — Essence of Linear Algebra (YouTube)** — watch fully first; then Gilbert Strang's MIT 18.06 for depth |
| Calculus: derivatives, chain rule, partial derivatives, gradients                  | Backpropagation is the chain rule applied recursively        | **3Blue1Brown — Essence of Calculus (YouTube)**; Khan Academy for practice                                           |
| Probability & statistics: Bayes theorem, distributions, expectation, variance, MLE | Model evaluation, A/B testing, probabilistic models          | **Statistics and Probability — Khan Academy** (free); **Think Stats — Allen Downey** (free PDF, code-first)          |
| Information theory: entropy, cross-entropy, KL divergence                          | Loss functions, model evaluation, compression                | Read the relevant chapters in Elements of Statistical Learning                                                       |

**Hands-on:**

- Implement matrix multiplication from scratch in numpy — no `@` operator. Then implement SVD decomposition and use it to compress an image. This makes embeddings intuitive.
- Implement gradient descent from scratch on a 2D loss surface, visualise the path with matplotlib. Then implement momentum and Adam. Run all three on the same surface and compare convergence.
- Simulate coin flips and compute a Bayesian posterior update by hand (and in numpy). Watch it converge as you add more data. This makes Bayesian A/B testing intuitive.

### Computer Science Foundations

| Topic                                                                         | Why it matters                                              | Resource                                                                               |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Big-O notation, time/space complexity                                         | Understand why ANN is O(log n) vs brute force O(n)          | **Grokking Algorithms — Aditya Bhargava** — visual, accessible                         |
| Hash maps, trees, heaps, graphs                                               | Used everywhere in pipeline and serving code                | **LeetCode Easy/Medium** — do 30–50 problems focused on arrays, hashmaps, trees        |
| Sorting and search algorithms                                                 | Interview staple; informs understanding of indexing         | Grokking Algorithms covers these                                                       |
| Networking basics: HTTP, REST, TCP/IP, latency vs throughput                  | Needed for serving, API design, debugging production issues | **Computer Networking: A Top-Down Approach** (or free Julia Evans zines on networking) |
| SQL fundamentals: joins, aggregations, window functions, indexes, query plans | Feature engineering, debugging pipelines                    | **Mode Analytics SQL Tutorial** (free); **SQLZoo** for practice                        |

**Hands-on:**

- Do LeetCode problems daily for 4–6 weeks — 2 Easy, 1 Medium per session. Focus on arrays, hashmaps, and sliding window patterns. Not for grinding interviews — for building pattern recognition.
- Write raw SQL to answer 10 business questions on a dataset you care about (e.g. your own transaction history exported to SQLite). Use window functions for at least 3 of them.
- Explain the query plan (`EXPLAIN ANALYZE`) for a slow SQL query and rewrite it to use an index. Do this in Postgres locally.

### Software Engineering Practices

| Topic                                                                     | Why it matters                                            | Resource                                               |
| ------------------------------------------------------------------------- | --------------------------------------------------------- | ------------------------------------------------------ |
| Git: branching, rebasing, conflict resolution, bisect                     | Daily use; bisect is a superpower for finding regressions | **Pro Git — Scott Chacon** (free online)               |
| Clean code: naming, single responsibility, avoiding premature abstraction | Production ML code is read far more than it's written     | **A Philosophy of Software Design — John Ousterhout**  |
| Testing: unit vs integration vs e2e, what to test and what not to         | Confidence in changes, catching regressions               | **Python Testing with pytest — Brian Okken**           |
| Shell / command line: pipes, grep, awk, sed, find, xargs                  | Debugging production jobs, data wrangling                 | **The Linux Command Line — William Shotts** (free PDF) |

**Hands-on:**

- Use `git bisect` to find which commit broke something in a test repo. Set it up yourself — introduce a bug, then find it with bisect.
- Refactor a 200-line function you've written into smaller pieces with clear names and no comments (the names should make comments unnecessary). Have someone else read it.
- Write a shell one-liner pipeline that: finds all `.log` files modified in the last 24h, extracts lines containing `ERROR`, counts occurrences by error type, and sorts descending. This forces fluency with pipes, grep, awk, sort, uniq.

---

## Learning Curriculum

Broken into skill domains. Each has what to learn, how deep to go, and where to learn it.

---

### 1. Python — Deep Internals

**What to learn:**

| Topic                                                                  | Depth                                               |
| ---------------------------------------------------------------------- | --------------------------------------------------- |
| Memory model: reference counting, `gc`, weak refs                      | Deep — understand when things get freed             |
| The GIL: what it blocks, what it doesn't                               | Deep — critical for concurrency decisions           |
| Generators / iterators: `yield`, `yield from`, `StopIteration`         | Deep — used everywhere in data pipelines            |
| Decorators: closures, `functools.wraps`, stacking                      | Deep                                                |
| Descriptors: `__get__`, `__set__`, `__delete__`                        | Medium — needed to understand property, classmethod |
| Metaclasses and `type`                                                 | Medium — understand it, don't overuse it            |
| `asyncio`: event loop, coroutines, `gather`, `TaskGroup`, backpressure | Deep — needed for any async serving code            |
| `dataclasses`, `__slots__`, `NamedTuple`                               | Deep — daily use                                    |
| Import system: `sys.modules`, `importlib`, circular imports            | Medium                                              |
| Profiling: `cProfile`, `line_profiler`, `py-spy`, `memory_profiler`    | Medium — know how to find bottlenecks               |

**Resources:**

- **Fluent Python, 2nd ed. — Luciano Ramalho** — read cover to cover, do the exercises
- **CPython Internals — Anthony Shaw** — walks through the actual CPython source
- **Python Cookbook, 3rd ed. — Beazley & Jones** — advanced recipes, great for descriptors/metaclasses
- [Real Python](https://realpython.com) — good for async and concurrency deep-dives
- [Python docs: Data Model](https://docs.python.org/3/reference/datamodel.html) — read this fully once

**Hands-on exercises:**

- **Build a lazy data pipeline** — implement a multi-stage generator pipeline that reads a large CSV line-by-line, filters, transforms, and aggregates without loading anything into memory. Profile memory usage with `memory_profiler` before and after.
- **Write a `@retry` decorator** from scratch — support configurable attempts, backoff strategy, and exception filtering. No libraries. Then write one that works on async functions too.
- **Implement `@cached_property`** using descriptors only — no `functools`. Write tests that confirm it only computes once and is instance-scoped.
- **Build a connection pool** using `asyncio.Queue` — a fixed-size pool of fake DB connections, async acquire/release, with a timeout. This forces you to understand the event loop properly.
- **Reproduce a memory leak** — write code that accidentally holds references (e.g. via a growing list inside a closure), use `gc` and `tracemalloc` to detect and fix it.
- **GIL experiment** — write a CPU-bound task, run it with threads vs processes, measure wall-clock time, explain the results in a comment.

---

### 2. Python Ecosystem & Tooling

**What to learn:**

| Tool               | What to understand                                                             |
| ------------------ | ------------------------------------------------------------------------------ |
| `uv`               | Dependency resolution, lockfiles, virtualenvs, workspace mode                  |
| `pyproject.toml`   | Full project config: build backend, tool config, metadata                      |
| `mypy` / `pyright` | Generics, `TypeVar`, `Protocol`, `Annotated`, strict mode                      |
| `pytest`           | Fixtures (scope, autouse), parametrize, monkeypatch, conftest                  |
| `hypothesis`       | Property-based testing — stateful testing is the hard part                     |
| `Pydantic v2`      | Validators, computed fields, custom types, model serialisation, `model_config` |
| `ruff`             | Linting + formatting — replaces flake8/black/isort                             |
| Pre-commit hooks   | Enforce standards at commit time                                               |

**Resources:**

- [uv docs](https://docs.astral.sh/uv/) — official, excellent
- [Pydantic v2 docs](https://docs.pydantic.dev/latest/) — work through all the validators section
- [mypy docs — type narrowing](https://mypy.readthedocs.io/en/stable/type_narrowing.html) — the non-obvious part
- [pytest docs — fixtures](https://docs.pytest.org/en/stable/how-to/fixtures.html)
- **Architecture Patterns with Python — Percival & Gregory** — DDD, ports/adapters in Python

**Hands-on exercises:**

- **Set up a new project from zero** using `uv init`, configure `pyproject.toml` with dev dependencies, ruff, mypy in strict mode. Make `uv run pytest` and `uv run mypy src/` both pass on a trivial module before writing any real code.
- **Type a messy existing script** — take one of your notebooks or scripts, convert it to a typed module, run `mypy --strict`, and fix every error. The errors you hit will teach you more than any tutorial.
- **Write a property-based test** with `hypothesis` for a pure function (e.g. a feature normalisation function). Make it find an edge case you hadn't considered.
- **Build a typed Pydantic v2 model** for a real data structure you work with (e.g. an expense event). Add custom validators, computed fields, and test serialisation to/from JSON. Add a `model_config` with `strict=True` and see what breaks.
- **Add pre-commit hooks** to a project — ruff, mypy, and a custom hook that blocks committing a file over 500 lines. Wire it up and confirm it blocks a bad commit.

---

### 3. Machine Learning — Fundamentals

Don't skip this. Understanding what's happening inside the models makes you a better engineer.

**What to learn:**

| Topic                                                               | Depth                                                    |
| ------------------------------------------------------------------- | -------------------------------------------------------- |
| Linear/logistic regression from scratch (numpy only)                | Deep — implement gradient descent manually               |
| Decision trees and how splitting works (Gini, entropy)              | Deep                                                     |
| Gradient boosting: how XGBoost/LightGBM differ from sklearn GBM     | Deep — know the leaf-wise vs level-wise split difference |
| Bias-variance tradeoff, overfitting, regularisation (L1/L2/dropout) | Deep                                                     |
| Cross-validation, leakage, proper train/val/test splits             | Deep — critical for any production model                 |
| Evaluation metrics: precision/recall/AUC/NDCG/MRR                   | Deep — NDCG and MRR are rec-system specific              |
| Dimensionality reduction: PCA, SVD                                  | Medium — needed to understand matrix factorisation       |
| Neural nets: forward pass, backprop, optimisers (SGD, Adam, AdamW)  | Deep if going into deep rec systems                      |

**Resources:**

- **Hands-On Machine Learning, 3rd ed. — Aurélien Géron** — the best practical ML book, do the code
- **The Elements of Statistical Learning — Hastie et al.** — free PDF, heavier math, reference text
- [fast.ai Practical Deep Learning](https://course.fast.ai/) — top-down approach, very code-heavy
- [StatQuest with Josh Starmer (YouTube)](https://www.youtube.com/@statquest) — best visual explanations of ML fundamentals
- **scikit-learn docs** — read the User Guide sections on estimators, pipelines, and model evaluation

**Hands-on exercises:**

- **Implement linear regression from scratch** using only numpy — gradient descent, no `sklearn`. Plot the loss curve. Then verify your weights match `sklearn.LinearRegression`. If they don't, find out why.
- **Implement a decision tree from scratch** — recursive splitting on Gini impurity, fit on a toy dataset, visualise the tree as text. This makes gradient boosting intuitive when you get there.
- **Reproduce a Kaggle tabular competition** end-to-end — pick any completed competition with structured data, download the data, engineer features, train XGBoost, and try to hit the top 25% leaderboard score. The feature engineering phase is the learning.
- **Deliberately cause and then detect data leakage** — build a dataset where the target leaks into a feature (e.g. include a future value), train a model, notice suspiciously good metrics, diagnose and fix it. Write up what you found.
- **Evaluate the same model with 5 different metrics** (accuracy, precision, recall, AUC-ROC, F1) on an imbalanced dataset. Write a one-paragraph explanation of which metric to trust and why — this is a common interview question.
- **Build a full sklearn Pipeline** — imputation, scaling, encoding, and a classifier all in one pipeline object. Add cross-validation. This is the correct way to avoid leakage in preprocessing.

---

### 4. Recommendation Systems

This is the specialist domain. Go deep here.

**What to learn:**

| Topic                                                                    | Depth                                    |
| ------------------------------------------------------------------------ | ---------------------------------------- |
| Collaborative filtering: user-based, item-based                          | Deep                                     |
| Matrix factorisation: ALS, SVD, implicit feedback                        | Deep — implement ALS from scratch once   |
| Two-tower neural models: query tower, item tower, dot-product similarity | Deep — dominant architecture in industry |
| Content-based filtering and hybrid models                                | Medium                                   |
| Candidate retrieval vs ranking — the two-stage funnel                    | Deep — most systems separate these       |
| Approximate nearest neighbour (ANN): HNSW, IVF, FAISS internals          | Deep — you'll use this constantly        |
| Evaluation: offline (NDCG, MAP, hit rate) vs online (CTR, conversion)    | Deep                                     |
| Cold start problem: new users, new items                                 | Medium                                   |
| Contextual bandits for exploration/exploitation                          | Medium                                   |
| Session-based recommendations (GRU4Rec, BERT4Rec concepts)               | Medium                                   |

**Resources:**

- **Practical Recommender Systems — Kim Falk** — best introductory practical book
- **Recommender Systems Handbook — Ricci et al.** — comprehensive academic reference
- [Eugene Yan's blog (eugeneyan.com)](https://eugeneyan.com) — best practitioner writing on rec systems in industry
- [Xavier Amatriain's rec systems course (Coursera/Stanford)](https://www.coursera.org/learn/recommender-systems) — solid foundations
- [Google's Recommendation Systems crash course](https://developers.google.com/machine-learning/recommendation) — free, well structured
- [FAISS documentation and wiki](https://github.com/facebookresearch/faiss/wiki) — read the index selection guide
- Papers to read:
  - _Deep Neural Networks for YouTube Recommendations_ (Covington et al., 2016)
  - _Wide & Deep Learning for Recommender Systems_ (Cheng et al., 2016)
  - _Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations_ (Yi et al., 2019)

**Hands-on exercises:**

- **Implement ALS from scratch** in numpy — just the alternating least squares update step, no Spark. Use the MovieLens 100K dataset (small enough). Verify your results roughly match `implicit` library output. This makes PySpark MLlib ALS non-magical.
- **Build a user-based and item-based CF system** from scratch — cosine similarity, top-N recommendations, no libraries. Measure NDCG@10 on a holdout set. Compare the two approaches.
- **FAISS index comparison exercise** — take 100K item embeddings, build three indices (Flat, IVF, HNSW), measure query time and recall@10 for each. Write up the tradeoffs in a README table. This is a real decision you'll make in production.
- **Implement NDCG from scratch** — write the formula by hand, verify against `sklearn.metrics.ndcg_score`. Then evaluate your ALS model with it. Understanding the metric deeply matters for rec system evaluation.
- **Cold start experiment** — in your MovieLens project, remove all training data for 20% of users, then try 3 strategies (popularity baseline, content-based, hybrid). Measure NDCG. Document which strategy wins and why.
- **Read and annotate the YouTube rec paper** — write a one-page summary of the two-stage architecture (retrieval → ranking). This paper describes what most production rec systems look like.

---

### 5. Data Engineering & Spark

Directly relevant to your current role — go deep.

**What to learn:**

| Topic                                                             | Depth                              |
| ----------------------------------------------------------------- | ---------------------------------- |
| Spark execution model: DAGs, stages, tasks, shuffle               | Deep — needed to debug performance |
| Spark SQL: window functions, lateral joins, aggregate pushdown    | Deep                               |
| Partitioning: how it affects performance, repartition vs coalesce | Deep                               |
| Delta Lake: ACID transactions, time travel, Z-ordering, OPTIMIZE  | Deep                               |
| Broadcast joins vs sort-merge joins — when to use each            | Deep                               |
| Catalyst optimiser: logical/physical plan, explain() output       | Medium                             |
| Spark streaming: structured streaming, watermarks, triggers       | Medium                             |
| Point-in-time correct feature engineering (no leakage)            | Deep — critical for ML pipelines   |
| Data skew: salting, adaptive query execution                      | Medium                             |

**Resources:**

- **Learning Spark, 2nd ed. — Damji et al.** — free PDF from Databricks, the standard text
- **Spark: The Definitive Guide — Chambers & Zaharia** — deeper than Learning Spark
- [Databricks Academy](https://www.databricks.com/learn/training) — free courses, directly applicable
- [Delta Lake documentation](https://docs.delta.io/latest/index.html) — read the internals sections
- [High Performance Spark — Karau & Warren] — performance tuning focused

**Hands-on exercises:**

- **Read and explain a query plan** — take any Spark job you've written, run `.explain(True)`, and write a plain-English explanation of every stage. Do this until you can spot a shuffle, a broadcast join, and a filter pushdown by eye.
- **Cause and fix a data skew** — create a synthetic dataset with 90% of records sharing one key, join it, observe the task time distribution in the Spark UI, then fix it with salting. Document before/after.
- **Window function drill** — write 10 different window function queries on a single dataset: running totals, rank, lag/lead, first/last, percentiles. Do this in both PySpark and Spark SQL. Window functions are asked about in every data engineering interview.
- **Point-in-time feature join** — create two tables: user events (with timestamps) and a slowly-changing feature table (e.g. user tier). Write a Spark job that joins them correctly so that for each event, you only use the feature value that was valid at that moment. This is harder than it sounds.
- **OPTIMIZE and Z-ORDER experiment** — take a Delta table, run a query on an unoptimised version, then run `OPTIMIZE` with `ZORDER BY` on your filter columns, re-run the query, compare scan metrics. Write up the result.
- **Tune a slow Spark job** — find or create a slow job (shuffle-heavy), use the Spark UI to diagnose it, and apply at least two optimisations (broadcast hint, repartition, persist, AQE settings). Measure improvement.

---

### 6. ML Engineering & MLOps

The gap between a data scientist and an ML engineer.

**What to learn:**

| Topic                                                                             | Depth                           |
| --------------------------------------------------------------------------------- | ------------------------------- |
| MLflow: tracking, model registry, serving, projects                               | Deep — know all four components |
| Feature stores: Feast architecture, point-in-time retrieval, online/offline split | Deep                            |
| Model serving patterns: batch scoring, real-time API, streaming                   | Deep                            |
| Train/serve skew — causes, detection, prevention                                  | Deep                            |
| Model versioning and rollback                                                     | Deep                            |
| Model monitoring: data drift, concept drift, performance degradation              | Deep                            |
| CI/CD for ML: training pipelines as code, automated retraining triggers           | Medium                          |
| Containerisation: Docker, Docker Compose, building lean images                    | Deep                            |
| REST API design for ML: async endpoints, batching, timeouts                       | Deep                            |

**Resources:**

- **Designing Machine Learning Systems — Chip Huyen** — the best book on production ML, read this first
- **Machine Learning Engineering — Andriy Burkov** — practical, concise
- [Made With ML (madewithml.com)](https://madewithml.com) — free MLOps course, very code-heavy
- [MLflow documentation](https://mlflow.org/docs/latest/index.html) — work through all tutorials
- [Feast documentation](https://docs.feast.dev) — architecture deep-dive section
- [Evidently documentation](https://docs.evidentlyai.com) — monitoring patterns

**Hands-on exercises:**

- **Log a full training run in MLflow** — not just metrics, but hyperparameters, the dataset version (hash or path), the git commit SHA, and the model artifact. Then load the model back from the registry and run inference. If you can't reproduce a run from the logged metadata alone, it's not logged correctly.
- **Reproduce train/serve skew deliberately** — train a model with one feature encoding, serve with a slightly different one, observe the prediction degradation. Then fix it by packaging the preprocessing with the model using an MLflow `PythonModel` wrapper.
- **Build a shadow deployment** — run two model versions in parallel, log both sets of predictions without serving the challenger to users, compare distributions using Evidently. This is how real teams test model changes safely.
- **Write a retraining trigger** — a script that queries recent prediction logs, computes PSI (Population Stability Index) on a key feature, and prints a warning if drift exceeds a threshold. No library for the PSI — implement the formula.
- **Containerise a model** — take a trained MLflow model, write a `Dockerfile` that installs deps and serves it via FastAPI, build and run it locally. Call it with `curl`. This is the minimum viable production deployment.
- **Design a feature store schema** for your current work domain — even on paper. Define which features are batch vs on-demand, what the entity keys are, and what the TTL should be. Implement it in Feast.

---

### 7. Deep Learning (for Rec Systems)

Learn this after you're solid on classical ML.

**What to learn:**

| Topic                                                                        | Depth  |
| ---------------------------------------------------------------------------- | ------ |
| PyTorch: tensors, autograd, custom datasets, DataLoader                      | Deep   |
| Embedding layers — what they are, how they're trained                        | Deep   |
| Two-tower architecture implementation in PyTorch                             | Deep   |
| Negative sampling strategies for implicit feedback                           | Deep   |
| Training at scale: mixed precision, gradient checkpointing, multi-GPU basics | Medium |
| ONNX export and model optimisation for serving                               | Medium |
| Transformers for sequences (concepts — BERT4Rec style)                       | Medium |

**Resources:**

- [fast.ai Part 1](https://course.fast.ai/) — best practical deep learning course
- **Deep Learning with PyTorch — Stevens, Antiga, Viehmann** — official PyTorch book
- [PyTorch documentation tutorials](https://pytorch.org/tutorials/) — work through custom datasets and training loops
- [Andrej Karpathy's Neural Networks: Zero to Hero (YouTube)](https://www.youtube.com/@AndrejKarpathy) — best from-scratch deep learning series available

**Hands-on exercises:**

- **Implement backpropagation from scratch** following Karpathy's `micrograd` — build a tiny autograd engine (Value class, `.backward()`), train a small MLP on XOR. Non-negotiable for understanding what PyTorch is doing.
- **Build a PyTorch training loop from scratch** — no Lightning, no Trainer abstraction. Dataset, DataLoader, model, optimiser, loss, train loop, validation loop, checkpointing. Do this once properly before using any framework.
- **Train an embedding model** — take item interaction data, define a simple matrix factorisation model in PyTorch (two embedding tables, dot product), train with BPR loss, extract item embeddings. Load them into FAISS and do similarity search.
- **Implement negative sampling** — in your embedding training, implement in-batch negatives (treat other items in the same batch as negatives). Understand why this is more efficient than random negatives and what the bias tradeoff is.
- **Compare embedding spaces** — train embeddings with and without negative sampling, with different dimensionalities (32, 64, 128). Visualise with UMAP or t-SNE. Write up what changes and why.
- **Export to ONNX and benchmark** — take a trained PyTorch model, export to ONNX, run inference with `onnxruntime`, measure latency vs PyTorch. This is the path to fast serving without GPU.

---

### 8. LLM Engineering & Agents

**What to learn:**

| Topic                                                              | Depth  |
| ------------------------------------------------------------------ | ------ |
| Prompt engineering: few-shot, chain-of-thought, structured outputs | Deep   |
| Function/tool calling: schema design, error handling               | Deep   |
| LangGraph: state machines, conditional edges, memory, subgraphs    | Deep   |
| Pydantic for structured LLM outputs                                | Deep   |
| RAG: chunking, embedding, retrieval, reranking                     | Deep   |
| Observability: tracing LLM calls, cost tracking, latency           | Deep   |
| Evals: how to test LLM behaviour systematically                    | Medium |

**Resources:**

- [Anthropic docs](https://docs.anthropic.com) — tool use and prompt engineering guides
- [LangGraph documentation](https://langchain-ai.github.io/langgraph/) — work through all the how-to guides
- [Langfuse documentation](https://langfuse.com/docs) — observability patterns
- [Eugene Yan on LLM evals](https://eugeneyan.com/writing/llm-evaluations/) — best practical writing on this

**Hands-on exercises:**

- **Build a structured extraction pipeline** — give an LLM a set of messy unstructured documents (receipts, emails, anything), define a Pydantic schema for the output, and use tool calling to force structured output. Measure extraction accuracy on 50 examples you've manually labelled.
- **Build a tool-using agent** from scratch using the Anthropic SDK directly — no LangGraph yet. Implement the tool call loop yourself: call API → check for tool use → execute tool → send result back → repeat. This makes LangGraph non-magical.
- **Reproduce a failure mode** — build an agent that sometimes loops, fails, or produces wrong output. Then add retry logic, output validation, and a fallback path. Observing failures firsthand is the fastest way to learn agent reliability patterns.
- **Add full observability** — instrument an agent with Langfuse so every LLM call, tool call, input, output, and latency is logged. Then use the traces to find the most expensive step and optimise it (prompt caching, smaller model for a sub-task, etc.).
- **Write an eval suite** for an LLM task — define 20 test cases with expected outputs, write an LLM-as-judge scorer, run it, and compute a score. Iterate on the prompt until the score improves. This is how you know your agent is getting better, not just different.
- **Build a RAG pipeline** — chunk a set of documents, embed them, store in Qdrant, retrieve on query, pass context to LLM. Then measure answer quality with and without retrieval on a set of questions. Add a reranker and measure again.

---

### 9. Interactive ML & Math Visualization

Build this alongside Month 1-2 — each visualization is a hands-on exercise for the fundamentals you're already implementing. The goal is not to build a polished product but to make abstract math tangible by implementing it yourself and seeing it move.

**What to learn:**

| Topic                                    | Why it matters                                                                   |
| ---------------------------------------- | -------------------------------------------------------------------------------- |
| Matplotlib animations (`FuncAnimation`)  | Animating gradient descent steps, weight updates                                 |
| Plotly 3D surfaces and interactive plots | Loss landscape visualization, parameter sweeps                                   |
| Streamlit                                | Instant interactive UI — sliders, buttons, live plots with no frontend knowledge |
| NumPy vectorized operations              | All the math runs through NumPy — matrix ops, eigenvectors, gradient computation |
| Numerical methods basics                 | Finite differences for derivatives, Euler integration for ODEs                   |

**Resources:**

- [Matplotlib animation docs](https://matplotlib.org/stable/api/animation_api.html) — `FuncAnimation` is all you need
- [Plotly Python docs](https://plotly.com/python/) — `go.Surface` for 3D loss landscapes
- [Streamlit docs](https://docs.streamlit.io) — get something interactive running in 30 minutes
- [3Blue1Brown — Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) — watch this first, then build the visualizations yourself
- [Seeing Theory](https://seeing-theory.brown.edu) — reference for what good probability visualizations look like

**Hands-on exercises:**

- **Animate gradient descent** — implement gradient descent from scratch on a 2D quadratic surface (`z = x² + y²`). Plot the loss surface with Plotly, animate the ball rolling to the minimum. Then try a non-convex surface with a saddle point and watch it get stuck.
- **Matrix transformation visualizer** — take the standard basis vectors [1,0] and [0,1], apply a 2x2 matrix transformation, animate the grid deforming. Then visualize what happens with a rotation matrix, shear, and projection matrix. This makes neural network layers intuitive.
- **Live neural network training** — train a small MLP (from scratch in NumPy) on a 2D classification dataset (two moons or concentric circles), update the decision boundary plot after each epoch using Streamlit. Add a slider for learning rate and watch training diverge when it's too high.
- **PCA from scratch** — take a 2D point cloud, compute the covariance matrix, find eigenvectors manually in NumPy (no `sklearn.decomposition`), plot the principal components as arrows on the data. Then extend to a 3D→2D projection.
- **Backpropagation step visualizer** — implement a 2-layer network and draw the computation graph as a diagram, highlighting which gradient is being computed at each step. Makes the chain rule concrete.

---

## Suggested Learning Order

```
Month 1-2:   Python internals (Fluent Python) + Hands-On ML (Géron) + Project 0 (Visualization)
Month 2-3:   Spark deep-dive (Learning Spark) + MLflow fundamentals
Month 3-4:   Rec systems foundations (Practical Recommender Systems + Google crash course)
Month 4-5:   Project 2 (Feature Pipeline) + Project 1 (Recommender)
Month 5-6:   PyTorch basics + two-tower models + Project 3 (Real-time ranking)
Month 6-7:   Designing ML Systems (Chip Huyen) + Project 6 (Mini ML Platform)
Month 7+:    LLM engineering + Project 4 + open source contributions
```

Papers are worth reading from month 4 onwards once you have the foundations.

---

## Project Roadmap

Projects are ordered by complexity. Each one builds on the last. Structure every project as a proper Python package — not a notebook dump.

```
my-project/
├── src/
│   └── myproject/
├── tests/
├── Makefile
├── pyproject.toml
└── README.md   ← explain design decisions, not just what it does
```

---

## Tier 1 — Foundation

### Project 0: ML Math Visualization Platform

**Goal:** Make the mathematical foundations of ML tangible by implementing them from scratch and visualizing them interactively. Build this alongside Month 1-2 — every visualization is a hands-on exercise you're already supposed to do, just with a visual layer on top.

**Why this project:** The full interactive platform in `interactive-math-physics-visualizations.md` is a multi-year effort. This is the scoped version — just the ML math visualizations that directly serve your learning roadmap right now. No C++/Rust/WASM needed yet. The learning value is in implementing the algorithms, not the rendering layer.

**Tech stack:** Python, NumPy, Matplotlib, Plotly, Streamlit. No frontend framework — Streamlit gives you interactive sliders and live plots in pure Python.

**What to build (in order):**

1. **Gradient Descent Visualizer**
   - Implement gradient descent from scratch in NumPy on a 2D loss surface
   - Plotly 3D surface plot with the optimization path animated step-by-step
   - Sliders for learning rate, starting position, number of steps
   - Add a non-convex surface with a saddle point — watch it get stuck
   - Extend to compare SGD, Momentum, and Adam on the same surface

2. **Matrix Transformation Visualizer**
   - Animate how a 2x2 matrix transforms the coordinate grid
   - Show rotation, shear, projection, and scaling matrices
   - Overlay eigenvectors — watch them stay on the same line while everything else rotates
   - Direct visual explanation of what neural network weight matrices do to data

3. **Neural Network Playground**
   - Train a small MLP from scratch in NumPy on 2D datasets (two moons, concentric circles, XOR)
   - Streamlit UI: paint your own dataset, adjust layers/neurons/learning rate with sliders
   - Decision boundary updates live after each epoch
   - Show training vs validation loss curves in real-time — deliberately cause overfitting

4. **PCA & SVD Visualizer**
   - Compute PCA from scratch (covariance matrix → eigenvectors, no sklearn)
   - Animate the principal components as arrows on a point cloud
   - Extend to 3D→2D projection — show which variance is preserved and which is lost
   - Connect to SVD: show image compression by dropping singular values

5. **Backpropagation Step Visualizer**
   - Draw the computation graph of a small network as a diagram
   - Step through the forward pass, then the backward pass, highlighting each gradient
   - Show the chain rule being applied at each node
   - Makes the connection between calculus and neural networks explicit

**Stretch goals (add after the core 5):**

- Fourier series epicycles (FFT-based curve drawing — beautiful and connects to signal processing)
- t-SNE running on real embeddings (MNIST or word vectors) with perplexity slider
- Bayesian posterior update animation (prior → likelihood → posterior as you add data points)

**Structure:**

```
ml-viz/
├── src/
│   └── ml_viz/
│       ├── gradient_descent.py    # NumPy implementations
│       ├── matrix_transforms.py
│       ├── neural_net.py
│       ├── pca_svd.py
│       └── backprop.py
├── apps/
│   ├── gradient_descent_app.py    # Streamlit app per visualization
│   ├── matrix_app.py
│   ├── neural_net_app.py
│   ├── pca_app.py
│   └── backprop_app.py
├── tests/
├── pyproject.toml
└── README.md
```

**Key constraint:** implement every algorithm from scratch in NumPy before building the visualization. The point is not a pretty UI — it's forcing you to implement gradient descent, eigenvectors, and backprop by hand. The visualization just makes it rewarding.

**Key skills:** NumPy, Matplotlib animations, Plotly 3D, Streamlit, numerical methods

---

### Project 1: End-to-end Recommender System

**Goal:** Touch every layer of a real recommendation pipeline.

**Dataset:** [MovieLens 25M](https://grouplens.org/datasets/movielens/25m/) — free, large enough to be non-trivial.

**What to build:**

1. Ingest raw interaction data, clean and store as Parquet/Delta
2. Train ALS collaborative filtering using **PySpark MLlib**
3. Build a FAISS index over item embeddings for approximate nearest neighbour search
4. Serve recommendations via a **FastAPI** endpoint
5. Track all experiments (hyperparams, metrics, model artifacts) with **MLflow**

**Stretch:** Replace ALS with a two-tower neural retrieval model in PyTorch. Train on GPU locally or via Google Colab.

**Key skills:** PySpark MLlib, FAISS, FastAPI, MLflow, Delta Lake

---

### Project 2: Feature Pipeline with a Feature Store

**Goal:** Most ML engineers skip this. Building it properly signals seniority.

**What to build:**

1. Pick a dataset with temporal structure — e.g. user purchase history or click logs
2. Engineer point-in-time correct features using Spark (no data leakage across the time axis)
3. Register and serve features via **Feast** (open source, self-hosted)
4. Train a gradient boosting model (XGBoost or LightGBM) consuming features from the store
5. Log the full training run in MLflow — feature versions, dataset snapshot, metrics

**Why it matters:** Feature stores enforce reproducibility and prevent train/serve skew — the #1 silent killer of real ML systems.

**Key skills:** Feast, PySpark, point-in-time joins, XGBoost/LightGBM, MLflow

---

## Tier 2 — Differentiation

### Project 3: Real-time Ranking System

**Goal:** Batch recommendations are table stakes. Real-time is where hard engineering lives.

**What to build:**

1. Simulate a stream of user events using **Kafka** (use Redpanda locally — same API, easier setup)
2. Compute rolling/windowed features in real-time using **Bytewax** (Python-native stream processing)
3. At request time: retrieve a candidate set from a vector DB (**Qdrant** — free to self-host), then re-rank using a lightweight model
4. Serve ranked results via FastAPI with sub-100ms p99 latency as a target

**Architecture:**

```
User event → Kafka → Bytewax (feature computation) → Redis (feature cache)
                                                           ↓
Request → FastAPI → Qdrant (retrieval) → re-ranker model → response
```

**Key skills:** Kafka/Redpanda, Bytewax, Qdrant, Redis, latency profiling

---

### Project 4: LLM-Powered Automation Agent

**Goal:** Combine your existing automations work with LLM engineering — highly employable combination.

**What to build:**

1. Pick a real workflow you find tedious (expense categorisation, document parsing, report generation are all directly relevant to your current role)
2. Build a multi-step agent using **LangGraph** with tools, memory, and conditional branching
3. Use **Pydantic v2** for all structured inputs/outputs — no raw string parsing
4. Log all traces and LLM calls with **Langfuse** (open source, self-hosted)
5. Expose as a REST API (FastAPI) or Slack bot

**Why it matters:** Agents that work reliably in production require proper state management, error recovery, and observability — LangGraph forces you to think in these terms.

**Key skills:** LangGraph, Anthropic SDK, Pydantic v2, Langfuse, FastAPI

---

### Project 5: A/B Testing Framework from Scratch

**Goal:** Understanding how ML decisions get validated in production. Almost nobody builds this themselves — it stands out.

**What to build:**

1. Implement a Bayesian A/B test engine in pure Python (no statsmodels shortcuts)
2. Simulate a recommendation system with two variants — different models or ranking strategies
3. Track conversion metrics per variant, compute credible intervals, implement early stopping logic
4. Build a **Streamlit** dashboard showing live experiment results and decision thresholds

**Why from scratch:** Using a library here misses the point. You need to understand when experiments are conclusive, not just how to call `.fit()`.

**Key skills:** Bayesian statistics, pure Python numerical code, Streamlit, experiment design

---

## Tier 3 — Senior-level Signal

### Project 6: Mini ML Platform

**Goal:** Shows systems thinking and infrastructure understanding, not just modelling ability.

**What to build:**

1. Feature computation job (PySpark) → writes to Delta table
2. Model training job (MLflow) → reads features, trains, registers model
3. FastAPI serving layer → loads model from registry, handles requests
4. Redis cache → low-latency feature lookups at serve time
5. Model monitoring → log predictions, detect distribution drift with **Evidently**
6. Containerise everything with **Docker Compose** — one command to bring it all up

**This is essentially a stripped-down version of what every serious ML team runs internally.**

**Key skills:** Docker Compose, MLflow model registry, Evidently, Redis, end-to-end system design

---

### Project 7: Open Source Contribution

**Goal:** Real code review from experienced engineers. Visible on GitHub permanently. Signals you can work in a real codebase.

**Good targets:**

- [MLflow](https://github.com/mlflow/mlflow) — experiment tracking, model registry
- [Feast](https://github.com/feast-dev/feast) — feature store
- [Evidently](https://github.com/evidentlyai/evidently) — ML monitoring
- [Bytewax](https://github.com/bytewax/bytewax) — stream processing in Python
- [LightFM](https://github.com/lyst/lightfm) — hybrid recommendation library

**How to start:** Fix a docs issue or add test coverage first to understand the codebase. Then pick a real `good first issue` bug.

---

## Full Tech Stack Reference

| Layer               | Primary Tools                                             |
| ------------------- | --------------------------------------------------------- |
| Data / storage      | PySpark, Delta Lake, Parquet                              |
| Feature engineering | Feast, point-in-time joins                                |
| Modelling           | scikit-learn, XGBoost, LightGBM, PyTorch                  |
| Recommendations     | FAISS, Qdrant, LightFM, ALS, two-tower                    |
| Serving             | FastAPI, Redis, Docker                                    |
| Experiment tracking | MLflow                                                    |
| LLM / agents        | LangGraph, Anthropic SDK, Langfuse                        |
| Streaming           | Kafka/Redpanda, Bytewax                                   |
| Monitoring          | Evidently                                                 |
| Visualisation       | Streamlit                                                 |
| Python tooling      | uv, pyproject.toml, mypy, pytest, hypothesis, Pydantic v2 |

---

## How to Work Through These

- **Don't use Jupyter for everything.** Notebooks are fine for exploration. For projects, structure as a proper Python package with `src/`, tests, and a `Makefile`. This alone separates your work from 80% of portfolio projects.
- **Write about the hard parts.** A README section on "why FAISS over exact search" or "how I handled train/serve skew" signals more than the code itself.
- **Recommended starting order given your current Databricks/automations background:**
  - Project 2 (Feature Pipeline) — directly extends what you already do
  - Project 1 (Recommender) — core rec systems foundations
  - Project 4 (LLM Agent) — extends your automations work
  - Then tier 2 and 3 from there
