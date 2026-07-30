---
layout: page
title: Recommendation Systems
permalink: /recommendation-systems/
---

The specialist domain. This is the primary focus — go deep here.

## Topics

| Topic | Depth |
|-------|-------|
| Collaborative filtering: user-based, item-based | Deep |
| Matrix factorisation: ALS, SVD, implicit feedback | Deep — implement ALS from scratch once |
| Two-tower neural models: query tower, item tower, dot-product similarity | Deep — dominant architecture in industry |
| Content-based filtering and hybrid models | Medium |
| Candidate retrieval vs ranking — the two-stage funnel | Deep — most production systems separate these |
| Approximate nearest neighbour (ANN): HNSW, IVF, FAISS internals | Deep — used constantly in production |
| Evaluation: offline (NDCG, MAP, hit rate) vs online (CTR, conversion) | Deep |
| Cold start problem: new users, new items | Medium |
| Contextual bandits for exploration/exploitation | Medium |
| Session-based recommendations: GRU4Rec, BERT4Rec concepts | Medium |

## Resources

- **Practical Recommender Systems — Kim Falk** — best introductory practical book
- **Recommender Systems Handbook — Ricci et al.** — comprehensive academic reference
- [Eugene Yan's blog (eugeneyan.com)](https://eugeneyan.com) — best practitioner writing on rec systems in industry
- [Google's Recommendation Systems crash course](https://developers.google.com/machine-learning/recommendation) — free, well structured
- [FAISS documentation and wiki](https://github.com/facebookresearch/faiss/wiki) — read the index selection guide

## Papers to Read

- *Deep Neural Networks for YouTube Recommendations* — Covington et al., 2016
- *Wide & Deep Learning for Recommender Systems* — Cheng et al., 2016
- *Sampling-Bias-Corrected Neural Modeling for Large Corpus Item Recommendations* — Yi et al., 2019

## Key Exercises

- Implement ALS from scratch in NumPy on MovieLens 100K — verify results roughly match `implicit` library output
- Build user-based and item-based CF from scratch — cosine similarity, top-N recommendations, measure NDCG@10 on a holdout set
- FAISS index comparison: Flat vs IVF vs HNSW on 100K embeddings — query time vs recall@10, write up tradeoffs
- Implement NDCG from scratch — verify against `sklearn.metrics.ndcg_score`, then evaluate your ALS model with it
- Cold start experiment: remove training data for 20% of users, compare popularity baseline vs content-based vs hybrid
- Read and annotate the YouTube rec paper — write a one-page summary of the two-stage retrieval → ranking architecture

---

*Notes and exercises will be added below as I work through this section.*
