---
layout: page
title: Visualization
permalink: /visualization/
---

Making abstract ML math tangible by implementing it from scratch and watching it move. Build alongside Month 1-2 — each visualization reinforces the fundamentals you're implementing in parallel.

The learning value is in implementing the algorithms, not the rendering layer.

## Topics

| Topic | Why It Matters |
|-------|---------------|
| Matplotlib `FuncAnimation` | Animating gradient descent steps, weight updates |
| Plotly 3D surfaces and interactive plots | Loss landscape visualisation, parameter sweeps |
| Streamlit | Instant interactive UI — sliders, live plots, no frontend knowledge needed |
| NumPy vectorized operations | All the math runs through NumPy |
| Numerical methods basics | Finite differences for derivatives, Euler integration |

## Resources

- [Matplotlib animation docs](https://matplotlib.org/stable/api/animation_api.html) — `FuncAnimation` is all you need
- [Plotly Python docs](https://plotly.com/python/) — `go.Surface` for 3D loss landscapes
- [Streamlit docs](https://docs.streamlit.io)
- [3Blue1Brown — Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) — watch first, then build the visualisations yourself
- [Seeing Theory](https://seeing-theory.brown.edu) — reference for what good probability visualisations look like

## Project 0: ML Math Visualization Platform

Build these in order — each one is a hands-on exercise for the fundamentals in the roadmap.

### 1. Gradient Descent Visualizer
- Implement gradient descent from scratch in NumPy on a 2D loss surface
- Plotly 3D surface plot with the optimisation path animated step-by-step
- Sliders for learning rate, starting position, number of steps
- Non-convex surface with a saddle point — watch it get stuck
- Compare SGD, Momentum, and Adam on the same surface

### 2. Matrix Transformation Visualizer
- Animate how a 2x2 matrix transforms the coordinate grid
- Show rotation, shear, projection, and scaling matrices
- Overlay eigenvectors — watch them stay on the same line while everything else rotates

### 3. Neural Network Playground
- Train a small MLP from scratch in NumPy on 2D datasets (two moons, concentric circles, XOR)
- Streamlit UI: paint your own dataset, adjust layers/neurons/learning rate with sliders
- Decision boundary updates live after each epoch — deliberately cause overfitting

### 4. PCA & SVD Visualizer
- Compute PCA from scratch (covariance matrix → eigenvectors, no sklearn)
- Animate principal components as arrows on a point cloud
- 3D→2D projection showing which variance is preserved and which is lost
- Connect to SVD: image compression by dropping singular values

### 5. Backpropagation Step Visualizer
- Draw the computation graph of a small network as a diagram
- Step through forward pass then backward pass, highlighting each gradient
- Show the chain rule being applied at each node

### Stretch Goals
- Fourier series epicycles (FFT-based curve drawing)
- t-SNE running on real embeddings (MNIST or word vectors) with perplexity slider
- Bayesian posterior update animation: prior → likelihood → posterior as data is added

---

*Notes and exercises will be added below as I work through this section.*
