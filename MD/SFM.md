# Structure from Motion (SfM)

```
Images
  ↓
Correspondence Search
  ├─ Feature Extraction
  ├─ Matching
  └─ Geometric Verification
  ↓
Incremental Reconstruction
  ├─ Initialization
  ├─ Image Registration  ↔  Outlier Filtering
  ├─ Triangulation
  └─ Bundle Adjustment
  ↓
Reconstruction (3D point cloud + camera poses)
```

Based on the full chat history and memory context, here are complete, cohesive notes on Structure from Motion:

# Structure from Motion (SfM)

## 1. What is SfM?

**Structure from Motion** recovers 3D scene geometry ("structure") and camera poses ("motion") from a set of 2D images. Formally:

> **Given** a set of corresponding points in two or more images, **compute** the camera parameters $(K, R, t)$ and the 3D point coordinates simultaneously.

It generalises stereo reconstruction to **unordered, uncalibrated, arbitrary-viewpoint** image collections.

## 2. The Projection Model

Every camera is modelled as a $3 \times 4$ **projection matrix**:

$$ \mathbf{P} = K[R \mid t] $$

| Symbol | Meaning | DOF |
|---|---|---|
| $K$ | $3 \times 3$ intrinsic matrix (focal length, principal point, skew) | 5 |
| $R$ | $3 \times 3$ rotation matrix ($R^T R = I,\ \det R = 1$) | 3 |
| $t$ | $3 \times 1$ translation vector | 3 |

A 3D point $\mathbf{X}_j$ projects to 2D pixel $\mathbf{x}_{ij}$ via:

$$ \mathbf{x}_{ij} \cong \mathbf{P}_i \mathbf{X}_j $$

The $\cong$ denotes equality up to scale (homogeneous coordinates). 

**Calibration info** = the intrinsic matrix $K$, determined once per camera by photographing a known checkerboard target — completely independent of the scene.

***

## 3. The SfM Problem: Formal Formulation

**Given:** $m$ images, $n$ fixed 3D points, $mn$ 2D observations $\mathbf{x}_{ij}$.

**Find:** All $m$ projection matrices $\mathbf{P}_i$ and all $n$ 3D points $\mathbf{X}_j$.

### Counting DOF

| Component | Count | Reason |
|---|---|---|
| Equations (knowns) | $2mn$ | Each 2D observation gives 2 scalar equations |
| Camera DOF (unknowns) | $11m$ | $3 \times 4 = 12$ entries minus 1 for scale |
| Point DOF (unknowns) | $3n$ | Each $(X, Y, Z)$ has 3 DOF |
| Projective ambiguity | $-15$ | Any $4 \times 4$ $Q$ (16 entries − 1 scale) is unresolvable |

### Solvability Constraint

$$ 2mn \geq 11m + 3n - 15 $$

For $m = 2$ cameras: substituting gives $n \geq 7$ — the **7-point minimum** for two views.

***

## 4. Reconstruction Ambiguities

Inserting $Q^{-1}Q = I$ into the projection equation:

$$ \mathbf{x} \cong \mathbf{P}\mathbf{X} = (\mathbf{P}Q^{-1})(Q\mathbf{X}) $$

The 2D image is **identical** for any $Q$. The type of $Q$ you can't resolve defines the ambiguity level:

### The Ambiguity Hierarchy

```
Projective Q (any 4×4)            ← raw matches only
    ↓  + parallelism constraints
Affine Q_A (bottom row = [0,0,0,1])
    ↓  + orthogonality / calibration (known K)
Similarity Q_S (sR in top-left)
    ↓  + known real-world distance
Euclidean (unique, s fixed)
```

### Detailed Breakdown

**Projective Ambiguity**
$Q$ = any full-rank $4 \times 4$ matrix. No constraints — cameras and scene can be arbitrarily warped. Only straight lines are preserved; parallel lines, angles, distances are all meaningless.

**Affine Ambiguity**
$$ Q_A = \begin{bmatrix} A & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix} $$
$A$ is a $3 \times 3$ full-rank matrix (9 DOF) — can shear and non-uniformly scale. The fixed bottom row $[\mathbf{0}^T\ 1]$ prevents perspective distortion, so **parallel lines stay parallel**. Angles and cross-direction length ratios are still lost — a cube can reconstruct as a slanted parallelepiped.

**Similarity Ambiguity**
$$ Q_S = \begin{bmatrix} s\mathbf{R} & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix} $$
Only an unknown global scale $s$ remains. Angles are correct, relative distances correct, parallelism preserved — the shape is exactly right, just unknown size. This is what calibrated SfM (known $K$) achieves.

**Euclidean Reconstruction**
$s$ is fixed by a known real-world distance reference. Everything is recovered. $R$ is constrained to 3 DOF (roll, pitch, yaw) — not 9 free entries.

| **Ambiguity** | **$Q$ form** | **DOF** | **Preserved** |
|---|---|---|---|
| Projective | Any $4 \times 4$ | 15 | Straight lines |
| Affine | $\begin{bmatrix} A & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 9 | Parallel lines |
| Similarity | $\begin{bmatrix} s\mathbf{R} & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 4 | Angles + shape |
| Euclidean | $\begin{bmatrix} R & \mathbf{t} \\ \mathbf{0}^T & 1 \end{bmatrix}$ | 3 | Everything |

***

## 5. COLMAP Pipeline Overview

$$ \text{Images} \rightarrow \underbrace{\text{Correspondence Search}}_{\text{Stage 1}} \rightarrow \underbrace{\text{Incremental Reconstruction}}_{\text{Stage 2}} \rightarrow \text{3D Model} $$

***

## 6. Stage 1 — Correspondence Search

### Feature Extraction
Run SIFT (or SURF/ORB/BRISK) on every image. Each keypoint has a **128-D descriptor** encoding local appearance. SIFT is scale and rotation invariant due to its DoG detector and orientation-normalised descriptor.

### Feature Matching
Compare descriptors across **every image pair** to find candidate matches. This builds a **scene graph** — nodes are images, edges connect pairs with verified matches. The image shows this as a network of Trevi Fountain photos with coloured dots marking matched keypoints.

### Geometric Verification
Raw matches contain many outliers. For each image pair, **RANSAC** estimates the **Fundamental Matrix** $F$, which encodes epipolar geometry:

$$ \mathbf{x}'^T F \mathbf{x} = 0 $$

Matches consistent with $F$ are kept as inliers; the rest are discarded. Only geometrically verified correspondences proceed to reconstruction.

***

## 7. Stage 2 — Incremental Reconstruction

### Initialization: Choosing the Seed Pair

The first image pair must satisfy two conflicting criteria:

| Pair Type | Matches | Baseline | Verdict |
|---|---|---|---|
| Very similar viewpoints | ✅ Many | ❌ Small | Bad — degenerate triangulation |
| Very different viewpoints | ❌ Few | ✅ Large | Bad — insufficient correspondences |
| **Ideal pair** | ✅ Many | ✅ Large | ✅ Use this |

**Why small baseline is bad:** The two projection rays are nearly parallel — a tiny pixel error shifts the 3D intersection point enormously along the ray. Like trying to locate a sound with ears 1mm apart.

**Why large baseline is bad:** The same physical point looks completely different from each camera — descriptors fail to match.

### From Seed Pair to Initial 3D — The Full Chain

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

**Scale note:** Recovery from $F$ alone gives only projective reconstruction. Known $K$ upgrades to similarity, and a physical reference upgrades to Euclidean.

### Image Registration (Adding New Cameras)

Each new image is registered by solving **Perspective-n-Point (PnP)**:
- **Input:** 2D feature detections in the new image + their corresponding already-triangulated 3D points (2D-3D correspondences)
- **Output:** Camera pose $(R, t)$ of the new camera; intrinsics $K$ estimated simultaneously for uncalibrated cameras
- **Robustness:** RANSAC wraps around PnP to reject outlier correspondences

### Next Best View Problem

Which unregistered image to add next matters significantly:

- **Too similar** to existing views → small baseline → high triangulation uncertainty
- **Too different** → low overlap → poor pose estimate; a single bad choice cascades errors

**Selection strategies:**
1. **Maximize visible triangulated points** — most 2D-3D correspondences → most stable PnP
2. **Minimize reconstruction uncertainty** — considers both count and spatial distribution of observations in the image (uniform spread constrains pose better than clustered points)

### Triangulation

Given two or more registered cameras with known $(K_i, R_i, t_i)$, and 2D observations of the same scene point, compute the 3D coordinates. Each camera contributes:

$$ \mathbf{x}_{ij} \cong \mathbf{P}_i \mathbf{X}_j $$

Rays from each camera should intersect — in practice they don't perfectly due to noise. 

Solve via **DLT**: reformulate as $A\mathbf{X} = 0$ and solve with SVD. Refine via geometric MLE (Levenberg-Marquardt) minimising reprojection error.

### Bundle Adjustment (BA)

BA jointly optimises **all** cameras and **all** 3D points simultaneously to minimise total reprojection error:

$$ \min_{\mathbf{P}_i,\ \mathbf{X}_j} \sum_{i=1}^{m} \sum_{j=1}^{n} w_{ij}\ d\!\left(\mathbf{x}_{ij},\ \text{proj}(\mathbf{P}_i \mathbf{X}_j)\right)^2 $$

| Symbol | Meaning |
|---|---|
| $w_{ij}$ | Binary visibility flag — is point $j$ visible in image $i$? |
| $d(\cdot)$ | Euclidean (L2) distance |
| $\mathbf{x}_{ij}$ | Observed 2D pixel location |
| $\text{proj}(\mathbf{P}_i \mathbf{X}_j)$ | Where the 3D point projects under current estimate |

**Solver:** The projection function is non-linear (homogeneous division), making this **non-linear least squares**. Solved by **Levenberg-Marquardt (LM)**:
- Far from minimum → behaves like gradient descent (cautious small steps)
- Near minimum → behaves like Gauss-Newton (fast, curvature-informed steps)
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
## 8. Global SfM (Alternative to Incremental)

Instead of adding images one by one, Global SfM estimates **all camera poses simultaneously**.

### Key Difference from Incremental

| Property | Incremental SfM | Global SfM |
|---|---|---|
| Pose estimation | One at a time | All cameras simultaneously |
| Error accumulation | Drift — errors compound | No drift — errors distributed globally |
| Bundle adjustment | Many small frequent BAs | One large well-initialised BA |
| Robustness | High (RANSAC per step) | Lower (bad pairs corrupt global solve) |
| Speed | Slower on large datasets | Faster — fewer BA iterations |

### Global SfM Pipeline

**Step 1 — Build Scene Graph:** Same correspondence search as incremental. Of all $\binom{N}{2}$ possible pairs, only $N_0$ pairs yield reliable $F$/$E$ estimates via RANSAC.

**Step 2 — Extract Relative Poses:** For each valid pair $(i,j)$, decompose $E$ into relative rotation $R_{ij}$ and translation direction $t_{ij}$.

**Step 3 — Rotation Averaging:** Recover global rotations $R_k$ for all $N$ cameras from the $N_0$ pairwise measurements. The consistency constraint is:

$$ R_{ij} = R_j R_i^T $$

This is overdetermined (more equations than unknowns) — solved via least-squares optimisation on the $SO(3)$ manifold. Crucially, **rotation can be solved independently of translation**.

**Step 4 — Translation Averaging:** With rotations known, recover global positions $T_k$. Each $t_{ij}$ is only a unit direction (scale lost from $E$ decomposition):

$$ t_{ij} \propto T_j - T_i $$

Direction constraints from all $N_0$ pairs are solved together for consistent global positions.

**Step 5 — Triangulate + Bundle Adjust:** With all cameras initialised globally, triangulate 3D points, then run one final BA pass. The well-initialised poses mean BA converges faster and more reliably.

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