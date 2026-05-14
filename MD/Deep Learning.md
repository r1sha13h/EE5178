# Deep Learning

## Three Key Ideas of Deep Learning

### 1. Hierarchical Compositionality
- Cascade of non-linear functions
- Multiple layers of abstraction
- Hidden layers increase abstraction

### 2. End-to-End Learning
- Learning (task-driven) representations
- Learning to extract features (no hand-crafted features)

### 3. Distributed Representation
- No single neuron encodes everything
- A group of neurons works together to represent a concept

## Universal Approximation Theorem

Simple neural networks can represent a wide variety of interesting functions given appropriate parameters.

**Practical consequence:** prefer more hidden layers over a single layer with a very large number of neurons.

## XOR Gate — Matrix Formulation

A 2-layer ReLU network solves XOR. Architecture:

$$\mathbf{y} = U^T \mathbf{h} + c, \qquad \mathbf{y} = U^T \underbrace{\max\{0,\, W\mathbf{x} + \mathbf{b}\}}_{\text{ReLU}} + c$$

$$\mathbf{h} = g(W\mathbf{x} + \mathbf{b}) = \max\{0,\, W\mathbf{x} + \mathbf{b}\}$$

**Parameters:**

$$W = \begin{bmatrix} 1 & 1 \\ 1 & 1 \end{bmatrix}, \quad \mathbf{b} = \begin{bmatrix} 0 \\ -1 \end{bmatrix}, \quad U = \begin{bmatrix} 1 \\ -2 \end{bmatrix}, \quad c = 0$$

**All four XOR inputs as columns of $X$:**

$$X = \begin{bmatrix} 0 & 1 & 0 & 1 \\ 0 & 0 & 1 & 1 \end{bmatrix}$$

**Layer 1 — affine:**

$$WX = \begin{bmatrix} 0 & 1 & 1 & 2 \\ 0 & 1 & 1 & 2 \end{bmatrix}, \qquad WX + \mathbf{b} = \begin{bmatrix} 0 & 1 & 1 & 2 \\ -1 & 0 & 0 & 1 \end{bmatrix}$$

**Apply ReLU:**

$$\mathbf{h} = \max\{0,\, WX + \mathbf{b}\} = \begin{bmatrix} 0 & 1 & 1 & 2 \\ 0 & 0 & 0 & 1 \end{bmatrix}$$

**Layer 2 output:** $\mathbf{y} = U^T\mathbf{h} = \begin{bmatrix} 0 & 1 & 1 & 0 \end{bmatrix}$ — the XOR truth table.

## KL Divergence

To compare two distributions $p$ and $q$:

$$D_{KL}(p \,\|\, q) = \sum_{x \in X} p(x)\, \ln\frac{p(x)}{q(x)}$$

Expanding the logarithm:

$$D_{KL}(p \,\|\, q) = \sum_x p(x)\ln p(x) - \sum_x p(x)\ln q(x)$$

$$D_{KL}(p \,\|\, q) = -H(p) + H(p, q)$$

where $H(p)$ is the **entropy** of $p$ and $H(p, q)$ is the **cross-entropy** between $p$ and $q$.
