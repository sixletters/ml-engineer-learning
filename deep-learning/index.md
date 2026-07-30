---
layout: page
title: Deep Learning
permalink: /deep-learning/
---

PyTorch from scratch, embedding models, and training for recommendation systems. Learn this after solid footing in classical ML.

## Topics

| Topic | Depth |
|-------|-------|
| PyTorch: tensors, autograd, custom datasets, DataLoader | Deep |
| Embedding layers — what they are, how they're trained | Deep |
| Two-tower architecture implementation in PyTorch | Deep |
| Negative sampling strategies for implicit feedback | Deep |
| Training at scale: mixed precision, gradient checkpointing, multi-GPU basics | Medium |
| ONNX export and model optimisation for serving | Medium |
| Transformers for sequences: BERT4Rec style concepts | Medium |

## Resources

- [fast.ai Part 1](https://course.fast.ai/) — best practical deep learning course
- **Deep Learning with PyTorch — Stevens, Antiga, Viehmann** — official PyTorch book
- [PyTorch tutorials](https://pytorch.org/tutorials/) — custom datasets and training loops
- [Andrej Karpathy — Neural Networks: Zero to Hero (YouTube)](https://www.youtube.com/@AndrejKarpathy) — best from-scratch deep learning series

## Key Exercises

- Implement backpropagation from scratch following Karpathy's `micrograd` — build a Value class with `.backward()`, train a small MLP on XOR
- Build a PyTorch training loop from scratch — no Lightning, no Trainer. Dataset, DataLoader, model, optimiser, loss, train loop, validation loop, checkpointing
- Train an embedding model: matrix factorisation in PyTorch with BPR loss, extract item embeddings, load into FAISS for similarity search
- Implement in-batch negative sampling — understand why it's more efficient than random negatives and what the bias tradeoff is
- Compare embedding spaces: train with different dimensionalities (32, 64, 128), visualise with UMAP or t-SNE, write up what changes
- Export to ONNX and benchmark: measure latency vs PyTorch inference — this is the path to fast serving without GPU

---

*Notes and exercises will be added below as I work through this section.*
