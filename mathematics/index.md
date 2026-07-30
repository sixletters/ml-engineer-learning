---
layout: page
title: Mathematics
permalink: /mathematics/
---

The math that underpins ML — linear algebra, calculus, probability, and information theory. Understanding this makes models non-magical and debugging intuitive.

## Topics

| Topic | Why It Matters |
|-------|---------------|
| Linear algebra: vectors, matrices, dot products, SVD, eigendecomposition | Embeddings, matrix factorisation, PCA are all linear algebra |
| Calculus: derivatives, chain rule, partial derivatives, gradients | Backpropagation is the chain rule applied recursively |
| Probability & statistics: Bayes theorem, distributions, expectation, variance, MLE | Model evaluation, A/B testing, probabilistic models |
| Information theory: entropy, cross-entropy, KL divergence | Loss functions, model evaluation, compression |

## Resources

- **3Blue1Brown — Essence of Linear Algebra (YouTube)** — watch fully first before anything else
- **Gilbert Strang's MIT 18.06** — for depth after 3B1B
- **3Blue1Brown — Essence of Calculus (YouTube)**
- **Statistics and Probability — Khan Academy** (free) + **Think Stats — Allen Downey** (free PDF, code-first)
- **Elements of Statistical Learning — Hastie et al.** (free PDF) — entropy and KL divergence chapters

## Key Exercises

- Implement matrix multiplication from scratch in NumPy (no `@` operator). Then implement SVD and use it to compress an image — this makes embeddings intuitive
- Implement gradient descent from scratch on a 2D loss surface, visualise the path with Matplotlib. Add momentum and Adam. Run all three on the same surface and compare convergence
- Simulate coin flips, compute a Bayesian posterior update by hand and in NumPy, watch it converge as data is added — makes Bayesian A/B testing intuitive

---

*Notes and exercises will be added below as I work through this section.*
