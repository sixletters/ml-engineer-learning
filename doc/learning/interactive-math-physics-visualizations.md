# Interactive Math, Physics & ML Visualization Platform

> An interactive educational platform for visualizing mathematical, physical, and machine learning concepts using WebAssembly (C++ + Rust hybrid) for high-performance computation and beautiful real-time rendering.

**Vision**: "3Blue1Brown meets interactive explorable explanations" - making advanced mathematics, physics, and machine learning tangible through interactive visualizations that run at native speeds in the browser.

## Why WebAssembly with C++ & Rust Hybrid?

- **Performance**: Heavy computations (neural networks, matrix operations, fluid dynamics, FFTs) run at near-native speed
- **C++ Strengths**: Leverage mature libraries (Eigen, FFTW, BLAS) for optimized linear algebra and numerical methods
- **Rust Strengths**: Memory safety for complex simulations, fearless concurrency, excellent WASM tooling
- **Real-time Interactivity**: Parameter tweaking without lag, smooth 60fps animations even with ML training
- **Complex Simulations**: Handle computationally expensive visualizations that pure JavaScript cannot
- **Best of Both Worlds**: C++ performance + Rust safety = robust, fast educational platform

---

## Mathematical Visualizations

### Calculus & Analysis
- **Derivative/Integral Visualizers**: Drag a function, see tangent lines and area under curve
- **Taylor Series Approximation**: Watch polynomials converge to sin(x), e^x, etc.
- **Gradient Descent**: Visualize optimization on 3D surfaces
- **Riemann Sums**: Watch numerical integration come alive
- **Limits & Continuity**: Interactive epsilon-delta proofs

### Linear Algebra
- **Matrix Transformations**: Watch the coordinate grid transform in real-time
- **Eigenvalue/Eigenvector Visualization**: See the special directions and scaling
- **SVD Decomposition**: Image compression visualized step-by-step
- **Linear Systems**: Visualize as intersecting planes in 3D
- **Gram-Schmidt Process**: Watch orthogonalization happen

### Complex Analysis
- **Domain Coloring**: Visualize complex functions f(z) with color mapping
- **Möbius Transformations**: Watch circles transform to circles
- **Conformal Mappings**: Angle-preserving transformations
- **Analyticity vs Singularities**: Visualize poles, branch cuts, essential singularities
- **Riemann Surfaces**: Multi-valued functions visualized

### Differential Equations
- **Phase Portraits**: Vector fields and trajectories
- **Interactive Systems**: Pendulums, springs, oscillators with parameter control
- **Predator-Prey Dynamics**: Lotka-Volterra equations visualized
- **Heat/Wave Equations**: PDE solvers with custom boundary conditions
- **Bifurcation Diagrams**: Watch stability changes as parameters vary

### Advanced Mathematical Topics
- **Fourier Analysis**: Epicycles drawing curves, signal decomposition, FFT visualization
- **Fractals & Chaos**: Mandelbrot deep zoom, Julia sets, bifurcation diagrams, strange attractors
- **Topology**: Knot theory, surface deformations, homeomorphisms
- **Numerical Methods**: Compare Euler vs RK4 vs adaptive steppers
- **Optimization**: Visualize gradient descent, Newton's method, constraint optimization

---

## Physics Simulations

### Classical Mechanics
- **Projectile Motion**: Interactive air resistance, drag coefficients, trajectory prediction
- **Pendulum Systems**: Double/triple pendulum chaos, coupled oscillators
- **Rigid Body Dynamics**: Colliding and rotating objects with realistic physics
- **Central Force Motion**: Orbital mechanics, Kepler's laws, gravitational systems
- **Lagrangian Mechanics**: Principle of least action, generalized coordinates
- **Hamiltonian Systems**: Phase space evolution, conservation laws

### Waves & Oscillations
- **Coupled Oscillators**: Wave propagation along chains
- **Interference Patterns**: Double-slit experiment, diffraction gratings
- **Standing Waves**: Strings, membranes, Chladni patterns
- **Doppler Effect**: Moving sources and observers
- **Wave Equation Solver**: 2D ripples with custom boundary conditions

### Thermodynamics & Statistical Mechanics
- **Molecular Dynamics**: Hard sphere gas, Maxwell-Boltzmann distribution emergence
- **Brownian Motion**: Random walks, diffusion
- **Ising Model**: Phase transitions, spontaneous magnetization, critical phenomena
- **Entropy Visualization**: Watch systems evolve toward equilibrium
- **Heat Diffusion**: Interactive heat sources and sinks

### Electromagnetism
- **Electric Fields**: Point charges, dipoles, conductors, field line visualization
- **Magnetic Fields**: Current loops, solenoids, magnetic materials
- **Particle in EM Field**: Cyclotron motion, E×B drift, charged particle trajectories
- **Electromagnetic Waves**: 3D wave propagation, polarization
- **Maxwell's Equations Solver**: Watch electric and magnetic fields evolve together

### Quantum Mechanics
- **Schrödinger Equation Solver**: Particle in a box, harmonic oscillator, quantum tunneling
- **Probability Density Evolution**: Watch wavefunctions evolve in time
- **Quantum Harmonic Oscillator**: Ladder operators, energy levels
- **Hydrogen Atom Orbitals**: Interactive 3D visualization of s, p, d, f orbitals
- **Quantum Spin**: Stern-Gerlach experiment, spin precession
- **Quantum Interference**: Double-slit with single particles

### Continuum Mechanics
- **Navier-Stokes Fluid Flow**: Smoke simulation, water, vortex dynamics
- **Elastic Deformation**: Stress/strain on beams, material properties
- **Acoustics**: Sound wave propagation, resonance
- **Lattice Boltzmann Method**: Alternative fluid simulation approach

### Relativity
- **Spacetime Diagrams**: Time dilation, length contraction, simultaneity
- **Geodesics**: Curved space visualization
- **Schwarzschild Metric**: Light bending near massive objects, gravitational lensing
- **Twin Paradox**: Worldlines in spacetime

---

## Machine Learning Mathematics

### Foundational Concepts

#### Linear Algebra for ML
- **Vector Spaces**: Visualize high-dimensional data projected to 2D/3D
- **Matrix Operations**: See how weight matrices transform data
- **Dot Products & Similarity**: Geometric interpretation of cosine similarity
- **Matrix Decompositions**: SVD, PCA visualized step-by-step
- **Eigenfaces**: Face recognition using eigenvectors
- **Linear Transformations**: How neural network layers transform data geometrically

#### Calculus for ML
- **Gradients in 2D/3D**: Vector fields showing direction of steepest ascent
- **Partial Derivatives**: Interactive surfaces with tangent planes
- **Chain Rule Visualization**: Backpropagation broken down step-by-step
- **Directional Derivatives**: How gradients point in parameter space
- **Jacobian & Hessian Matrices**: Second-order optimization insights

#### Probability & Statistics
- **Probability Distributions**: Interactive PDFs and CDFs (Normal, Bernoulli, etc.)
- **Central Limit Theorem**: Watch distributions converge
- **Bayes' Theorem**: Visual updates of prior → posterior
- **Maximum Likelihood Estimation**: Likelihood surfaces
- **Confidence Intervals**: Bootstrap and Monte Carlo visualization
- **Conditional Probability**: Venn diagrams and decision trees

### Neural Networks & Deep Learning

#### Network Architecture
- **Neuron Activation**: Single neuron with adjustable weights and bias
- **Layer Transformations**: Watch data flow through network layers
- **Activation Functions**: Compare sigmoid, tanh, ReLU, softmax visually
- **Universal Approximation**: Watch network approximate arbitrary functions
- **Decision Boundaries**: 2D classification with interactive dataset painting
- **Network Topology**: Visualize feedforward, CNN, RNN, Transformer architectures

#### Training Dynamics
- **Backpropagation**: Step-through animation showing gradient flow
- **Loss Surfaces**: 3D visualization of loss landscapes
- **Gradient Descent Variants**: Compare SGD, Momentum, Adam, RMSprop paths
- **Learning Rate Effects**: Too small (slow), too large (divergence), just right
- **Batch Size Impact**: Mini-batch vs full batch gradient paths
- **Overfitting vs Underfitting**: Training/validation curves in real-time
- **Regularization**: L1/L2 effects on weight distributions
- **Dropout**: Watch neurons randomly deactivate during training

#### Loss Functions & Optimization
- **Loss Landscapes**: Interactive 3D surfaces (convex, non-convex, saddle points)
- **Gradient Descent Path**: Trace optimization through parameter space
- **Local Minima vs Global Minima**: Visualize optimization challenges
- **Momentum**: Ball rolling down loss surface with velocity
- **Adaptive Learning Rates**: Watch step sizes adjust
- **Optimization Algorithms Race**: Multiple optimizers solving same problem
- **Gradient Flow**: Vector field visualization in weight space

### Classical Machine Learning

#### Linear Models
- **Linear Regression**: Interactive line fitting with residuals
- **Polynomial Regression**: Degree selection and overfitting
- **Logistic Regression**: Decision boundary and sigmoid transformation
- **Ridge & Lasso**: Regularization paths and coefficient shrinkage

#### Support Vector Machines
- **Maximum Margin**: Interactive hyperplane with support vectors
- **Kernel Trick**: Data transformation to higher dimensions
- **Kernel Comparison**: Linear, RBF, polynomial kernels side-by-side
- **Soft Margin**: Slack variables and C parameter effects

#### Decision Trees & Ensembles
- **Decision Tree Growth**: Watch tree split data interactively
- **Information Gain**: Entropy and Gini impurity visualized
- **Random Forest**: Multiple trees voting
- **Gradient Boosting**: Sequential error correction visualization
- **Feature Importance**: Bar charts updating during training

#### Clustering
- **K-Means**: Watch centroids move and clusters form
- **DBSCAN**: Density-based clustering with epsilon/min_points
- **Hierarchical Clustering**: Dendrogram building animation
- **Gaussian Mixture Models**: EM algorithm convergence
- **Cluster Validation**: Silhouette scores, elbow method

### Dimensionality Reduction

- **PCA**: Watch principal components emerge from data cloud
- **t-SNE**: High-dimensional data spiraling into 2D clusters
- **UMAP**: Manifold learning with adjustable parameters
- **Autoencoders**: Encode → compress → decode visualization
- **Feature Maps**: CNN activations and learned filters
- **Embedding Spaces**: Word2Vec, GloVe vector arithmetic (king - man + woman = queen)

### Advanced ML Concepts

#### Attention Mechanisms
- **Self-Attention**: Query-key-value matrix operations visualized
- **Multi-Head Attention**: Parallel attention heads
- **Attention Weights**: Heatmap of which inputs attend to which
- **Transformer Blocks**: Data flow through complete architecture

#### Generative Models
- **GANs**: Generator vs Discriminator adversarial training
- **VAE Latent Space**: Interactive 2D latent space exploration
- **Diffusion Models**: Denoise process step-by-step
- **Distribution Matching**: Watch generator distribution approach real data

#### Reinforcement Learning
- **Q-Learning**: Value function heatmaps on gridworld
- **Policy Gradients**: Policy distribution evolution
- **Bellman Equation**: Value propagation through states
- **Exploration vs Exploitation**: Epsilon-greedy, UCB visualized
- **Actor-Critic**: Separate policy and value networks

### Information Theory & ML

- **Entropy**: Visualize information content of distributions
- **KL Divergence**: Distance between distributions
- **Mutual Information**: Shared information visualization
- **Cross-Entropy Loss**: Decomposed into entropy + KL divergence
- **Information Bottleneck**: Compression vs prediction tradeoff

### Statistical Learning Theory

- **Bias-Variance Tradeoff**: Interactive model complexity slider
- **VC Dimension**: Shattering and model capacity
- **PAC Learning**: Sample complexity bounds
- **ROC Curves**: TPR vs FPR with threshold slider
- **Precision-Recall**: Tradeoff for imbalanced datasets
- **Confusion Matrix**: Interactive with classification threshold

### Practical ML Visualizations

#### Data Preprocessing
- **Normalization**: Watch data scale to [0,1] or standard normal
- **Missing Data Imputation**: Mean, median, KNN strategies
- **Feature Scaling**: Compare different scaling methods
- **Outlier Detection**: IQR, Z-score, isolation forest

#### Model Evaluation
- **Cross-Validation**: K-fold splits visualized
- **Learning Curves**: Sample size vs performance
- **Validation Curves**: Hyperparameter vs performance
- **Residual Plots**: Regression diagnostics
- **Calibration Curves**: Predicted vs actual probabilities

#### Feature Engineering
- **Polynomial Features**: Interaction terms visualization
- **Binning**: Continuous to categorical transformation
- **One-Hot Encoding**: Categorical expansion
- **Feature Crosses**: Combination features in 2D/3D

---

## C++ & Rust Hybrid Architecture

Given the computational demands of ML math, physics simulations, and advanced visualizations, we'll leverage both C++ and Rust:

### Why Both Languages?

**C++ Strengths:**
- Mature ML/math libraries (Eigen, BLAS, LAPACK, Armadillo)
- Highly optimized linear algebra operations
- FFT libraries (FFTW)
- Existing neural network frameworks (tiny-dnn, dlib)

**Rust Strengths:**
- Memory safety for complex state management
- Modern development experience with excellent tooling
- Superior WebAssembly integration (wasm-pack, wasm-bindgen)
- Fearless concurrency for parallel simulations
- No data races in multi-threaded ML training

### Recommended Project Structure

```
project/
├── rust-core/                  # Main Rust implementation
│   ├── src/
│   │   ├── lib.rs             # WASM entry points
│   │   ├── bridge.rs          # C++/Rust interop (cxx bridge)
│   │   ├── ml/
│   │   │   ├── neural_net.rs  # Neural network training loops
│   │   │   ├── optimizers.rs  # SGD, Adam, etc.
│   │   │   └── datasets.rs    # Data handling
│   │   ├── physics/
│   │   │   ├── fluids.rs      # Navier-Stokes solver
│   │   │   ├── particles.rs   # N-body simulations
│   │   │   └── mechanics.rs   # Classical mechanics
│   │   └── math/
│   │       ├── calculus.rs    # Numerical derivatives
│   │       └── statistics.rs  # Statistical functions
│   │
│   ├── cpp/                    # C++ high-performance kernels
│   │   ├── eigen_wrapper.cpp  # Linear algebra via Eigen
│   │   ├── fft_wrapper.cpp    # Fourier transforms via FFTW
│   │   ├── blas_ops.cpp       # Optimized BLAS operations
│   │   └── ml_kernels.cpp     # Convolution, pooling ops
│   │
│   ├── build.rs               # Rust build script (compiles C++)
│   └── Cargo.toml
│
├── wasm-core/                  # Pure C++ modules (if needed)
│   ├── geometry/              # Computational geometry (CGAL)
│   └── special_functions/     # Bessel, Gamma functions
│
├── web-ui/                     # TypeScript/React frontend
│   ├── src/
│   │   ├── visualizations/
│   │   │   ├── ml/            # ML visualizations
│   │   │   ├── math/          # Math visualizations
│   │   │   └── physics/       # Physics simulations
│   │   ├── components/        # Shared UI components
│   │   └── workers/           # Web Workers for threading
│   └── public/
│
└── docs/                       # Documentation
```

### C++/Rust Interop Strategy

Use the **`cxx` crate** for safe, zero-cost interop:

```rust
// rust-core/src/bridge.rs
#[cxx::bridge]
mod ffi {
    unsafe extern "C++" {
        include!("cpp/eigen_wrapper.h");
        
        // C++ matrix operations
        fn matrix_multiply(a: &[f64], b: &[f64], m: usize, n: usize, k: usize) -> Vec<f64>;
        fn svd_decompose(matrix: &[f64], rows: usize, cols: usize) -> Vec<f64>;
        fn fft_1d(signal: &[f64]) -> Vec<f64>;
        
        // Neural network kernels
        fn conv2d(input: &[f64], kernel: &[f64], params: ConvParams) -> Vec<f64>;
        fn batch_norm(input: &[f64], mean: f64, variance: f64) -> Vec<f64>;
    }
    
    extern "Rust" {
        // Rust functions callable from C++
        fn train_neural_network(data: &[f64], labels: &[u32], epochs: u32) -> Vec<f64>;
    }
}

// Use C++ for heavy lifting, Rust for orchestration
pub fn train_model(dataset: &Dataset) -> Model {
    let mut weights = initialize_weights();
    
    for epoch in 0..num_epochs {
        // Use C++ Eigen for fast matrix ops
        let activations = ffi::matrix_multiply(&dataset.x, &weights, m, n, k);
        
        // Use Rust for safe gradient computation and updates
        let gradients = compute_gradients_safe(&activations, &dataset.y);
        update_weights(&mut weights, &gradients);
    }
    
    Model { weights }
}
```

---

## Tech Stack

### Core Computation

**Rust (Primary)**
- **ndarray**: N-dimensional arrays (like NumPy)
- **nalgebra**: Linear algebra library
- **wasm-bindgen**: Seamless JS/WASM interop
- **wasm-pack**: Build tool for WASM projects
- **rayon**: Data parallelism (compiles to Web Workers)
- **serde**: Serialization for data transfer

**C++ (High-Performance Kernels)**
- **Eigen**: Industry-standard linear algebra
- **FFTW**: Fastest Fourier Transform in the West
- **BLAS/LAPACK**: Optimized matrix operations
- **Armadillo**: High-level matrix library
- **tiny-dnn**: Lightweight neural networks
- **Emscripten**: C++ to WebAssembly compilation

**Interop**
- **cxx**: Safe C++/Rust bidirectional bindings
- **cbindgen**: Auto-generate C++ headers from Rust

### Frontend
- **WebGL/WebGPU**: GPU-accelerated rendering
- **Canvas2D**: Clean 2D diagrams and plots
- **MathJax/KaTeX**: Beautiful equation rendering
- **React + TypeScript**: Component-based UI
- **Plotly.js / D3.js**: Interactive charts and graphs
- **Three.js**: 3D visualization framework

### Build & Deployment
- **wasm-pack**: Rust → WASM build pipeline
- **Emscripten**: C++ → WASM compilation
- **Vite**: Fast frontend bundler with WASM support
- **Web Workers**: Multi-threaded computation in browser

---

## Implementation Priority: "Greatest Hits"

Start with these showcase visualizations to prove the concept:

### 1. **Neural Network Playground** ⭐⭐
- Interactive neural network training on 2D datasets
- Paint your own dataset, watch decision boundary evolve
- Adjust layers, learning rate, activation functions in real-time
- **ML**: Backpropagation, gradient descent, overfitting
- **Visual Impact**: Very High
- **Educational Value**: Exceptional (demystifies deep learning)

### 2. **Gradient Descent on Loss Landscapes** ⭐⭐
- 3D loss surface with multiple optimizers racing
- Compare SGD, Momentum, Adam, RMSprop
- Interactive: place ball anywhere, watch it roll to minimum
- **ML/Math**: Optimization, calculus, momentum
- **Visual Impact**: Very High
- **Why**: Makes abstract optimization concrete

### 3. **Fourier Series Epicycles** ⭐
- Draw any shape, decompose into rotating circles
- Beautiful, mesmerizing, educational
- **Math**: FFT, complex analysis, signal processing
- **Visual Impact**: High
- **Connection to ML**: Basis for signal processing in CNNs

### 4. **Double Pendulum Chaos** ⭐
- Interactive double pendulum with trail visualization
- Demonstrates sensitivity to initial conditions
- **Physics**: Classical mechanics, chaos theory
- **Visual Impact**: High

### 5. **Linear Transformations & PCA** ⭐
- Watch matrix operations transform the coordinate grid
- Overlay PCA: see principal components emerge from data
- **Math/ML**: Linear algebra, dimensionality reduction
- **Educational Value**: Very High
- **Connection**: Foundation of neural network layers

### 6. **Fluid Simulation** ⭐
- Real-time Navier-Stokes or Lattice Boltzmann
- Interactive obstacle drawing, smoke/dye injection
- **Physics**: Continuum mechanics, PDEs
- **Visual Impact**: Very High

### 7. **t-SNE Dimensionality Reduction** ⭐
- Watch high-dimensional data (MNIST, embeddings) collapse to 2D
- Interactive perplexity slider shows clustering changes
- **ML**: Manifold learning, embeddings
- **Visual Impact**: High (mesmerizing cluster formation)

### 8. **Fractal Deep Zoom**
- Mandelbrot/Julia sets with arbitrary precision
- Smooth zooming to extreme magnifications
- **Math**: Complex analysis, numerical precision
- **Visual Impact**: Very High

---

## Features

### Interactive Controls
- Real-time parameter sliders (mass, friction, initial conditions, etc.)
- Equation input with live parsing
- Preset configurations and examples
- Reset and randomize options

### Educational Elements
- Side-by-side: equations ↔ visualizations
- Step-by-step mode for algorithms
- Tooltips explaining physics/math concepts
- Links to further reading and theory

### Sharing & Persistence
- URL-based configuration sharing
- Save/load custom scenarios
- Export animations as video/GIF
- Screenshot capture

### Performance Optimization
- Adaptive quality based on device capability
- Multi-threaded computation (Web Workers)
- Efficient rendering with requestAnimationFrame
- Memory management and object pooling

---

## Use Cases

1. **ML Education**: Learn machine learning math by seeing it (backprop, gradient descent, attention)
2. **Mathematics Learning**: Interactive calculus, linear algebra, complex analysis
3. **Physics Education**: Classical mechanics, quantum mechanics, electromagnetism
4. **Teaching**: Professors creating interactive lecture materials
5. **Research**: Scientists prototyping simulations and visualizing results
6. **Data Science**: Understand algorithms before applying to real data
7. **Self-Learning**: Math/physics/ML enthusiasts exploring concepts interactively
8. **Technical Interviews**: Visualize algorithm complexity and optimization
9. **Art & Creativity**: Generative artists using mathematical systems

---

## Future Enhancements

- **VR/AR**: Immersive 3D mathematical spaces
- **Collaborative Mode**: Multi-user exploration
- **Programming Interface**: Let users write custom simulations
- **Mobile Optimization**: Touch-friendly controls
- **Educational Paths**: Guided learning sequences
- **Community Gallery**: User-created visualizations

---

## Getting Started

### Development Roadmap

**Phase 1: Foundation & Interop**
- Set up Rust + C++ hybrid build pipeline (wasm-pack + cxx)
- Implement C++ Eigen wrapper for linear algebra
- Create Rust WASM entry points with wasm-bindgen
- Basic 2D Canvas/WebGL rendering
- Test C++/Rust interop with simple matrix operations

**Phase 2: Core "Greatest Hits" Visualizations**
- **Neural Network Playground**: Interactive training on 2D data
- **Gradient Descent Visualizer**: 3D loss landscapes
- **Linear Transformations**: Matrix operations on grids
- **Fourier Epicycles**: FFT-based curve drawing
- Develop reusable UI components (sliders, controls, equation renderer)

**Phase 3: Expand ML & Math**
- PCA, t-SNE dimensionality reduction
- Backpropagation step-by-step visualizer
- Calculus visualizations (derivatives, integrals)
- Probability distributions (interactive PDFs/CDFs)
- Decision trees, SVMs, clustering algorithms

**Phase 4: Physics Simulations**
- Double pendulum chaos
- Fluid dynamics (Navier-Stokes)
- N-body gravity simulation
- Quantum mechanics (Schrödinger solver)
- Wave simulations

**Phase 5: Advanced Features**
- Dataset upload & custom visualization
- Real-time performance profiling
- Educational tooltips & guided tours
- Save/share configurations via URL
- Export animations

**Phase 6: Polish & Deploy**
- Performance optimization (SIMD, multi-threading)
- Mobile/touch support
- Comprehensive documentation
- Video tutorials
- Community examples gallery

---

## Example Learning Journeys

### Journey 1: Understanding Neural Networks from Scratch
1. **Linear Transformations** → See how matrices transform space
2. **Gradient Descent** → Understand optimization visually
3. **Single Neuron** → Weighted sum + activation function
4. **Neural Network Playground** → Train 2-layer network on custom data
5. **Backpropagation Visualizer** → Watch gradients flow backward
6. **Loss Landscapes** → See why we get stuck in local minima
7. **Overfitting Demo** → Balance model complexity vs generalization

### Journey 2: Linear Algebra for ML
1. **Vector Spaces** → Visualize high-dimensional data
2. **Matrix Multiplication** → See transformations compose
3. **Eigenvalues/Eigenvectors** → Find special directions
4. **SVD** → Decompose matrices, compress images
5. **PCA** → Dimensionality reduction on real data
6. **t-SNE** → Non-linear manifold learning

### Journey 3: Calculus for Deep Learning
1. **Derivatives** → Rate of change visualized
2. **Gradients** → Direction of steepest ascent
3. **Chain Rule** → Composition of functions (key to backprop)
4. **Partial Derivatives** → Multi-variable optimization
5. **Taylor Series** → How neural networks approximate functions
6. **Optimization Algorithms** → Compare different descent strategies

### Journey 4: Physics Meets ML
1. **Classical Mechanics** → Equations of motion
2. **Lagrangian Mechanics** → Principle of least action (connects to optimization)
3. **Statistical Mechanics** → Probability distributions emerge
4. **Boltzmann Machines** → Physics-inspired neural networks
5. **Hamiltonian Neural Networks** → Energy-conserving ML models
6. **Diffusion Models** → Thermodynamics in generative AI

---

## Inspiration & References

**Mathematics & Physics**
- **3Blue1Brown**: Mathematical visualization excellence (linear algebra, calculus)
- **Explorable Explanations**: Interactive learning paradigm (Bret Victor)
- **PhET Interactive Simulations**: Physics and chemistry simulations
- **Desmos**: Beautiful graphing calculator
- **GeoGebra**: Dynamic mathematics software

**Machine Learning**
- **TensorFlow Playground**: Neural network visualization (inspiration for our NN playground)
- **Distill.pub**: Beautiful ML research communication
- **Seeing Theory**: Visual introduction to probability and statistics
- **MLU Explain**: Amazon's ML concept visualizations
- **ConvNetJS**: Deep learning in the browser
- **GAN Lab**: Interactive GAN visualization

**Technical Resources**
- **Eigen Documentation**: C++ linear algebra library
- **WebAssembly Documentation**: WASM specs and best practices
- **cxx crate**: Safe C++/Rust interop patterns
- **wasm-bindgen**: Rust/JavaScript interop guide

---

**This platform aims to make advanced mathematics, physics, and machine learning not just understandable, but *experienceable* - turning abstract concepts into interactive playgrounds where curiosity drives learning and intuition emerges from exploration.**
