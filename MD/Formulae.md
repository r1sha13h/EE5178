## Epipolar Geometry

$$\mathbf{l}' = F\mathbf{x}$$

$$\boxed{\mathbf{x}'^T F \mathbf{x} = 0}$$

$$P = K[I\,|\,\mathbf{0}], \qquad P' = K'[R\,|\,\mathbf{t}]$$

where K, K′ are intrinsic matrices, R is rotation, **t** is translation from camera 1 to camera 2.

The 3D point at depth $z$ on the ray through pixel **x** is:

$$\mathbf{X}(z) = \begin{pmatrix} z K^{-1}\mathbf{x} \\ 1 \end{pmatrix}$$

$$\mathbf{l}' = \mathbf{p} \times \mathbf{q} = (K'\mathbf{t}) \times (K'RK^{-1}\mathbf{x})$$

Apply the identity $(M\mathbf{a}) \times (M\mathbf{b}) = M^{-T}(\mathbf{a} \times \mathbf{b})$:

$$\mathbf{l}' = K'^{-T}\bigl(\mathbf{t} \times (RK^{-1}\mathbf{x})\bigr) = K'^{-T}[\mathbf{t}]_\times R K^{-1}\mathbf{x}$$

Therefore:

$$\boxed{F = K'^{-T}[\mathbf{t}]_\times R K^{-1}}$$

$$[\mathbf{t}]_\times = \begin{bmatrix} 0 & -t_3 & t_2 \\ t_3 & 0 & -t_1 \\ -t_2 & t_1 & 0 \end{bmatrix}, \qquad \mathbf{t} \times \mathbf{x} = [\mathbf{t}]_\times \mathbf{x}$$

### 6.1 Parallel (Side-by-Side) Stereo Rig

Cameras face the same direction, separated horizontally:

$$K = K' = \begin{bmatrix}f&0&0\\0&f&0\\0&0&1\end{bmatrix}, \quad R = I, \quad \mathbf{t} = \begin{pmatrix}t_x\\0\\0\end{pmatrix}$$

Computing $F = K'^{-T}[\mathbf{t}]_\times R K^{-1}$:

$$F = \frac{t_x}{f^2}\begin{bmatrix}0&0&0\\0&0&-1\\0&1&0\end{bmatrix} \;\sim\; \begin{bmatrix}0&0&0\\0&0&-1\\0&1&0\end{bmatrix}$$

### 6.2 Forward Translating Camera

Camera moves forward along its optical axis:

$$K = K' = \begin{bmatrix}f&0&0\\0&f&0\\0&0&1\end{bmatrix}, \quad R = I, \quad \mathbf{t} = \begin{pmatrix}0\\0\\t_z\end{pmatrix}$$

$$F = \frac{t_z}{f^2}\begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix} \;\sim\; \begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix}$$

### Fundamental Matrix Estimation

$$\min_{\mathbf{f}} \|A\mathbf{f}\|^2 \quad \text{subject to} \quad \|\mathbf{f}\|^2 = 1$$

**Solution: SVD of A.**

$$A = U\Sigma V^T$$

### Key Equations
| Equation | Meaning |
|---|---|
| $\mathbf{x}'^T F \mathbf{x} = 0$ | Fundamental epipolar constraint |
| $\mathbf{l}' = F\mathbf{x}$ | Epipolar line in image 2 for point **x** in image 1 |
| $\mathbf{l} = F^T\mathbf{x}'$ | Epipolar line in image 1 for point **x′** in image 2 |
| $F = K'^{-T}[\mathbf{t}]_\times R K^{-1}$ | F computed from known camera matrices |
| $F\mathbf{e} = \mathbf{0}$ | **e** is the right null vector of F |
| $F^T\mathbf{e}' = \mathbf{0}$ | **e′** is the left null vector of F |

## Stereo Reconstruction

### Limitations of similarity constraint:
- Textureless surfaces
- Repetitive patterns
- Occlusions
- Non-Lambertian surfaces & Specularities
- Foreshortening effects

### Additional Constraints for Disambiguation
- Uniqueness
- Ordering
- Smoothness of disparity field


### Disparity and Depth
$$\boxed{d = x - x' = \frac{bf}{Z}}$$

### The Energy Function

$$E = \alpha \, E_{\text{data}}(I_1, I_2, D) + \beta \, E_{\text{smooth}}(D)$$

**Data term** — photometric consistency:
$$E_{\text{data}} = \sum_i \bigl(W_1(i) - W_2(i + D(i))\bigr)^2$$
Penalizes large differences between matched patches.

**Smoothness term** — spatial regularization:
$$E_{\text{smooth}} = \sum_{\text{neighbors } i,j} \rho\bigl(D(i) - D(j)\bigr)$$

### Triangulation

### Method-1

#### Expanding with Camera Rows
Writing $\mathbf{P}$ row-by-row as $\mathbf{p}_1^\top, \mathbf{p}_2^\top, \mathbf{p}_3^\top$:

$$\begin{bmatrix} y\mathbf{p}_3^\top - \mathbf{p}_2^\top \\ \mathbf{p}_1^\top - x\mathbf{p}_3^\top \\ x\mathbf{p}_2^\top - y\mathbf{p}_1^\top \end{bmatrix}\mathbf{X} = \mathbf{0}$$

#### Full System from Both Cameras

$$\underbrace{\begin{bmatrix} y\mathbf{p}_3^\top - \mathbf{p}_2^\top \\ \mathbf{p}_1^\top - x\mathbf{p}_3^\top \\ y'\mathbf{p}_3'^\top - \mathbf{p}_2'^\top \\ \mathbf{p}_1'^\top - x'\mathbf{p}_3'^\top \end{bmatrix}}_{\mathbf{A} \ (4\times4)} \mathbf{X} = \mathbf{0}$$

### Method-2

Find $\hat{\mathbf{X}}$ that exactly satisfies camera geometry:
$$\hat{\mathbf{x}} = \mathbf{P}\hat{\mathbf{X}}, \qquad \hat{\mathbf{x}}' = \mathbf{P}'\hat{\mathbf{X}}$$
minimizing the **reprojection error**

$$\min_{\hat{\mathbf{X}}} \ \mathcal{C}(\mathbf{x}, \mathbf{x}') = d(\mathbf{x}, \hat{\mathbf{x}})^2 + d(\mathbf{x}', \hat{\mathbf{x}}')^2$$

#### Statistical Justification
If measurement noise is Gaussian $\sim \mathcal{N}(0, \sigma^2)$, then:
$$p(\mathbf{x} \mid \hat{\mathbf{x}}) \propto \exp\!\left(-\frac{d(\mathbf{x}, \hat{\mathbf{x}})^2}{2\sigma^2}\right)$$

## SFM

### Types of Ambiguity

| **Ambiguity** | **$Q$ form** | **DOF** | **Preserved** |
|---|---|---|---|
| Projective | Any $4 \times 4$ | 15 | Straight lines |
| Affine | $\begin{bmatrix} A & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 9 | Parallel lines |
| Similarity | $\begin{bmatrix} s\mathbf{R} & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 4 | Angles + shape |
| Euclidean | $\begin{bmatrix} R & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 3 | Everything |

### Triangulation Process

```
Matched 2D points (x ↔ x')
    ↓
8-point algorithm + RANSAC  →  Fundamental Matrix F
    ↓
F × intrinsics K             →  Essential Matrix E = K'^T F K
    ↓
SVD decomposition             →  4 candidate (R, t) pairs
    ↓
Cheirality check              →  Correct (R, t) [points in front of both cameras]
    ↓
Camera 1: P₁ = K[I | 0]      (world origin)
Camera 2: P₂ = K[R | t]      (recovered pose)
    ↓
DLT Triangulation             →  Initial sparse 3D point cloud
```
### Bundle Adjustment (BA)

BA jointly optimises **all** cameras and **all** 3D points simultaneously to minimise total reprojection error:

$$ \min_{\mathbf{P}_i,\ \mathbf{X}_j} \sum_{i=1}^{m} \sum_{j=1}^{n} w_{ij}\ d\!\left(\mathbf{x}_{ij},\ \text{proj}(\mathbf{P}_i \mathbf{X}_j)\right)^2 $$

**Levenberg-Marquardt (LM)**:
- Far from minimum → gradient descent
- Near minimum → Gauss-Newton
- A damping parameter $\lambda$ controls the switch dynamically

### The Full Incremental Loop

```
1. Pick best seed pair
   → 8-point + RANSAC → F → E → (R,t) → triangulate initial points

2. For each new image:
   a. Match features to existing 3D points (2D-3D correspondences)
   b. PnP + RANSAC → new camera pose
   c. Triangulate new 3D points
   d. Re-optimise existing points visible in new image
   e. Bundle Adjustment → refine ALL cameras + ALL 3D points
   f. Outlier filtering → remove high-reprojection-error points

3. Repeat until all images registered
```

### Global SfM

```
N images
  ↓
Correspondence search → scene graph (N₀ valid pairs)
  ↓
E decomposition → R_ij, t_ij per pair
  ↓
Rotation averaging → global R_k
  ↓
Translation averaging → global T_k
  ↓
Triangulation → 3D point cloud
  ↓
Bundle Adjustment (one pass) → final reconstruction
```