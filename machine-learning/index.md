---
layout: page
title: Machine Learning
permalink: /machine-learning/
---

Classical ML fundamentals. Don't skip this — understanding what's happening inside models makes you a better engineer when things break in production.

## Topics

| Topic | Depth |
|-------|-------|
| Linear/logistic regression from scratch (NumPy only) | Deep — implement gradient descent manually |
| Decision trees: splitting, Gini impurity, entropy | Deep |
| Gradient boosting: XGBoost/LightGBM vs sklearn GBM | Deep — leaf-wise vs level-wise splits |
| Bias-variance tradeoff, overfitting, regularisation (L1/L2/dropout) | Deep |
| Cross-validation, leakage, proper train/val/test splits | Deep — critical for any production model |
| Evaluation: precision, recall, AUC-ROC, NDCG, MRR | Deep — NDCG and MRR are rec-system specific |
| Dimensionality reduction: PCA, SVD | Medium — needed to understand matrix factorisation |
| Neural nets: forward pass, backprop, optimisers (SGD, Adam, AdamW) | Deep |

## Resources

- **Hands-On Machine Learning, 3rd ed. — Aurélien Géron** — the best practical ML book, do the code
- **Elements of Statistical Learning — Hastie et al.** — heavier math, reference text (free PDF)
- [fast.ai Practical Deep Learning](https://course.fast.ai/) — top-down, very code-heavy
- [StatQuest with Josh Starmer (YouTube)](https://www.youtube.com/@statquest) — best visual explanations of ML fundamentals
- **scikit-learn docs** — User Guide sections on estimators, pipelines, and model evaluation

## Key Exercises

- Implement linear regression from scratch in NumPy — gradient descent, no `sklearn`. Plot loss curve. Verify weights match `sklearn.LinearRegression`
- Implement a decision tree from scratch — recursive splitting on Gini impurity, fit on a toy dataset, visualise as text
- Reproduce a Kaggle tabular competition end-to-end — download data, engineer features, train XGBoost, aim for top 25%
- Deliberately cause data leakage — notice suspiciously good metrics, diagnose and fix it. Write up what you found
- Evaluate the same model with accuracy, precision, recall, AUC-ROC, and F1 on an imbalanced dataset — explain which to trust and why
- Build a full sklearn `Pipeline`: imputation + scaling + encoding + classifier + cross-validation

---

*Notes and exercises will be added below as I work through this section.*
