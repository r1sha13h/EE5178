***
# Epipolar Geometry — Complete Notes
**Course:** EE5178 — Modern Computer Vision
*Adapted from lectures by Rob Fergus (NYU) and Ioannis Gkioulekas (CMU)*

***
## 1. Motivation: Stereo Reconstruction
The goal of two-view geometry is to recover **3D structure and shape** from two images of the same scene. Given two cameras viewing the same scene, the output is the 3D positions of scene points — a process called **stereo reconstruction**.

The two images can arise from:
- A **stereo rig** — two cameras capturing the scene simultaneously
- A **single moving camera** — two frames of a static scene captured sequentially

These scenarios are **geometrically equivalent**. The basic reconstruction principle is **triangulation**: find the 3D point at the intersection of the two back-projected rays, one from each camera.

**Two-step algorithm:**
1. For each point in image 1, find the corresponding point in image 2 — *(search problem)*
2. For each matched pair, compute the 3D point by triangulation — *(estimation problem)*

Finding correspondences naively is a **2D search** across the entire second image. Epipolar geometry reduces this to a **1D search along a line**, called the epipolar line.

***
## 2. Notation
Two cameras **P** and **P′** project 3D point **X** to image points:

$$\mathbf{x} = P\mathbf{X}, \qquad \mathbf{x}' = P'\mathbf{X}$$

- **P**, **P′** — 3×4 camera matrices
- **X** — homogeneous 4-vector (3D point)
- **x**, **x′** — homogeneous 3-vectors (2D image points)

> **Convention:** All equations with homogeneous coordinates hold *up to scale* — equality means equal up to a non-zero scalar multiple.

***
## 3. Epipolar Geometry
### 3.1 Core Idea
Given a point **x** in image 1, the corresponding 3D scene point lies somewhere along the ray back-projected through **x**. This entire ray, projected into image 2, sweeps out a **line** — called the **epipolar line l′**. The corresponding point **x′** must lie somewhere on this line.

**Epipolar constraint:** Reduces correspondence search from 2D → **1D along the epipolar line.**
### 3.2 Key Definitions
| Term | Definition |
|---|---|
| **Epipole e** | Projection of camera 2's centre C′ into image 1; equivalently, where the baseline pierces image 1 |
| **Epipole e′** | Projection of camera 1's centre C into image 2 |
| **Baseline** | Line joining the two camera centres C and C′ |
| **Epipolar line l′** | Line in image 2 corresponding to point **x** in image 1 |
| **Epipolar plane** | Plane containing C, C′, and the 3D point X — these three points are always coplanar |
| **Epipolar pencil** | One-parameter family of epipolar planes as X varies; all epipolar lines in each image converge at the epipole |
### 3.3 Image Centre vs. Epipole
These are two different concepts that happen to coincide in one special case:

- **Image centre (principal point):** Where the optical axis hits the sensor — fixed hardware property, always at roughly the geometric centre of the image.
- **Epipole:** Projection of the *other camera's centre* — can be anywhere, including outside the image boundary.

They coincide only for **pure forward translation**, where the baseline is aligned with the optical axis (Section 6.2).
### 3.4 The Epipolar Pencil
As 3D point X moves along the back-projected ray, the epipolar plane **rotates about the baseline**, generating a family of planes — the **epipolar pencil**. Two critical consequences:
- Every epipolar line in image 1 passes through **e**
- Every epipolar line in image 2 passes through **e′**

***
## 4. The Fundamental Matrix
### 4.1 Algebraic Setup
The mapping $\mathbf{x} \rightarrow \mathbf{l}'$ depends only on the cameras P and P′, not on scene structure. It is **linear** and encoded by a single 3×3 matrix **F**:

$$\mathbf{l}' = F\mathbf{x}$$

If **x** and **x′** are corresponding points, then **x′** lies on **l′**, giving:

$$\boxed{\mathbf{x}'^T F \mathbf{x} = 0}$$

This is the **fundamental epipolar constraint** — the central equation of two-view geometry.
### 4.2 Derivation of F (Three Steps)
Choose cameras with the first camera at the world origin:

$$P = K[I\,|\,\mathbf{0}], \qquad P' = K'[R\,|\,\mathbf{t}]$$

where K, K′ are intrinsic matrices, R is rotation, **t** is translation from camera 1 to camera 2.

***

**Step 1 — Back-project a ray from x using K⁻¹**

The 3D point at depth $z$ on the ray through pixel **x** is:

$$\mathbf{X}(z) = \begin{pmatrix} z K^{-1}\mathbf{x} \\ 1 \end{pmatrix}$$

Applying K⁻¹ removes intrinsic camera distortion (focal length, pixel scaling) and recovers the pure 3D direction. Think of K as the map from 3D directions → pixels; K⁻¹ reverses it.

***

**Step 2 — Pick two convenient points on the ray and project into image 2**

Rather than tracking a general depth-z point, pick the two most convenient points:

| Point on ray | Depth | 3D form | Projects to image 2 as |
|---|---|---|---|
| Camera centre C | $z=0$ | $(\mathbf{0},\,1)^T$ | $\mathbf{p} = K'\mathbf{t}$ — this is the **epipole e′** |
| Point at infinity | $z=\infty$ | $(K^{-1}\mathbf{x},\,0)^T$ | $\mathbf{q} = K'RK^{-1}\mathbf{x}$ |

The camera centre of camera 1, viewed from camera 2, is always the epipole — this is the geometric definition of e′. The point at infinity on the ray carries the direction information.

***

**Step 3 — Compute line through p and q using the cross product**

The line through two 2D points is their cross product:

$$\mathbf{l}' = \mathbf{p} \times \mathbf{q} = (K'\mathbf{t}) \times (K'RK^{-1}\mathbf{x})$$

Apply the identity $(M\mathbf{a}) \times (M\mathbf{b}) = M^{-T}(\mathbf{a} \times \mathbf{b})$:

$$\mathbf{l}' = K'^{-T}\bigl(\mathbf{t} \times (RK^{-1}\mathbf{x})\bigr) = K'^{-T}[\mathbf{t}]_\times R K^{-1}\mathbf{x}$$

Therefore:

$$\boxed{F = K'^{-T}[\mathbf{t}]_\times R K^{-1}}$$

**Pipeline in one line:**

$$\underbrace{\mathbf{x}}_{\text{pixel in cam 1}} \xrightarrow{\;K^{-1}\;} \underbrace{\text{3D ray direction}} \xrightarrow{\;R,\,\mathbf{t}\;} \underbrace{\text{2 projected pts in cam 2}} \xrightarrow{\;\times\;} \underbrace{\mathbf{l}' = F\mathbf{x}}_{\text{epipolar line in cam 2}}$$
### 4.3 The Cross-Product Matrix
The vector cross product $\mathbf{v} \times \mathbf{x}$ is equivalent to a matrix multiplication:

$$[\mathbf{v}]_\times = \begin{bmatrix} 0 & -v_3 & v_2 \\ v_3 & 0 & -v_1 \\ -v_2 & v_1 & 0 \end{bmatrix}, \qquad \mathbf{v} \times \mathbf{x} = [\mathbf{v}]_\times \mathbf{x}$$

Key property: $[\mathbf{v}]_\times \mathbf{v} = \mathbf{0}$ (self-annihilation). Also, $\det[\mathbf{v}]_\times = 0$ always — so it is **always rank 2** for any non-zero **v**.

***
## 5. Properties of the Fundamental Matrix
### 5.1 Full Summary Table
| Property | Statement |
|---|---|
| **Rank** | Rank 2; $\det F = 0$ |
| **DOF** | 7 degrees of freedom |
| **Scale** | Homogeneous — defined only up to a scalar multiple |
| **Point correspondence** | $\mathbf{x}'^T F \mathbf{x} = 0$ for all corresponding pairs |
| **Epipolar line in image 2** | $\mathbf{l}' = F\mathbf{x}$ |
| **Epipolar line in image 1** | $\mathbf{l} = F^T\mathbf{x}'$ |
| **Epipoles** | $F\mathbf{e} = \mathbf{0}$ and $F^T\mathbf{e}' = \mathbf{0}$ |
| **Transpose** | $F^T$ is the fundamental matrix for the reversed camera pair (P′, P) |
### 5.2 Direction of Mapping
F acts as a bridge from camera 1 to camera 2. The output line **always lives in the other camera's image**, not the one where the input point came from:

| Input | Mapping | Output |
|---|---|---|
| $\mathbf{x}$ in camera 1 | $F$ | $\mathbf{l}' = F\mathbf{x}$ in camera 2 |
| $\mathbf{x}'$ in camera 2 | $F^T$ | $\mathbf{l} = F^T\mathbf{x}'$ in camera 1 |

The prime (′) consistently denotes camera 2.
### 5.3 Why F Must Be Rank 2 — Three Independent Proofs
**Proof 1 — Geometric:**
F maps points to *lines*, not to arbitrary 3-vectors. A line in homogeneous 2D has 2 degrees of freedom. Rank 2 constrains F's output to a 2D subspace — exactly the space of lines. A rank-3 matrix would allow any arbitrary 3-vector as output, which has no geometric meaning as a line.

**Proof 2 — Null space:**
The epipole satisfies $F\mathbf{e} = \mathbf{0}$ (proven below). A non-zero vector mapping to zero means F has a **non-trivial null space** — only possible if rank < 3. Since exactly one epipole exists, the null space is 1-dimensional, forcing rank exactly 2. If rank were 1, the null space would be 2-dimensional — too many epipoles.

**Proof 3 — From the formula:**
In $F = K'^{-T}[\mathbf{t}]_\times R K^{-1}$, the matrix $[\mathbf{t}]_\times$ is always rank 2 (its determinant is identically zero for any **t**). Surrounding full-rank matrices K′, R, K cannot increase rank beyond the minimum:

$$\text{rank}(F) \leq \text{rank}([\mathbf{t}]_\times) = 2$$
### 5.4 Why $F\mathbf{e} = \mathbf{0}$ and $F^T\mathbf{e}' = \mathbf{0}$
**This is not assumed — it is derived from geometry.**

Every epipolar line $\mathbf{l}' = F\mathbf{x}$ passes through **e′** (because every epipolar line in image 2 converges at e′). So:

$$\mathbf{e}'^T \mathbf{l}' = \mathbf{e}'^T (F\mathbf{x}) = 0 \quad \text{for EVERY possible } \mathbf{x}$$

The only way a dot product equals zero for **every** possible **x** is if the multiplier is the zero vector:

$$\mathbf{e}'^T F = \mathbf{0} \;\implies\; F^T\mathbf{e}' = \mathbf{0}$$

By symmetry (swapping camera roles): $F\mathbf{e} = \mathbf{0}$.

**Logical chain:**

1. All epipolar lines $\mathbf{l}'$ pass through $\mathbf{e}'$.
2. Therefore $\mathbf{e}'^T(F\mathbf{x}) = 0$ for all $\mathbf{x}$.
3. Hence $\mathbf{e}'^T F = \mathbf{0}$, so $F^T\mathbf{e}' = \mathbf{0}$.
4. By symmetry, $F\mathbf{e} = \mathbf{0}$.

**Geometric intuition:** The epipole **e** in camera 1 is the projection of the baseline itself. Back-projecting the ray through **e** recovers the baseline — which passes through camera 2's own centre. A line passing through a camera centre projects to the entire image plane, not a single line — hence **e** cannot be assigned a unique epipolar line. Mathematically, "no unique output" = zero vector = null space.
***
## 6. Special Cases
### 6.1 Parallel (Side-by-Side) Stereo Rig
Cameras face the same direction, separated horizontally:

$$K = K' = \begin{bmatrix}f&0&0\\0&f&0\\0&0&1\end{bmatrix}, \quad R = I, \quad \mathbf{t} = \begin{pmatrix}t_x\\0\\0\end{pmatrix}$$

Computing $F = K'^{-T}[\mathbf{t}]_\times R K^{-1}$:

$$F = \frac{t_x}{f^2}\begin{bmatrix}0&0&0\\0&0&-1\\0&1&0\end{bmatrix} \;\sim\; \begin{bmatrix}0&0&0\\0&0&-1\\0&1&0\end{bmatrix}$$

The scalar $t_x/f^2$ is dropped because F is homogeneous (scale-invariant). The constraint $\mathbf{x}'^T F \mathbf{x} = 0$ reduces to $y = y'$ — **epipolar lines are horizontal scan lines**. The epipole is at infinity in the direction $(1,0,0)^T$ — off-screen — as expected when both cameras face the same direction and can never "see" each other's centre.
### 6.2 Forward Translating Camera
Camera moves forward along its optical axis:

$$K = K' = \begin{bmatrix}f&0&0\\0&f&0\\0&0&1\end{bmatrix}, \quad R = I, \quad \mathbf{t} = \begin{pmatrix}0\\0\\t_z\end{pmatrix}$$

$$F = \frac{t_z}{f^2}\begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix} \;\sim\; \begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix}$$

Note: $t_z$ (magnitude of translation) and $f^2$ (pixel-scaling) are both scalars that multiply all entries uniformly. Since F is homogeneous, they carry no geometric information and cancel.

The epipolar line for any point $\mathbf{x} = (x, y, 1)^T$:

$$\mathbf{l}' = F\mathbf{x} = \begin{pmatrix}-y \\ x \\ 0\end{pmatrix}$$

The third component being 0 means the line **passes through the origin**. The constraint $-yx' + xy' = 0$ gives $y'/x' = y/x$ — a **radial line emanating from (0,0)**.

**Verifying epipole lies on l′:**
$$(-y)(0) + (x)(0) + (0)(1) = 0 \;\checkmark$$

The epipole $(0,0,1)^T$ is the **image centre** — which lies on every radial line.

**Physical meaning:** Moving forward causes all scene objects to flow outward from a central point — the **focus of expansion**. That point is the epipole. Every epipolar line is a radial spoke from the centre. This is exactly what you see driving through a tunnel or flying through space — all motion streaks radiate outward from one point straight ahead.
### 6.3 Why F Has Constant Values Despite Variable t
Both examples show F with entries like 0, 1, −1 — no $t_x$ or $t_z$ visible. This is because **F is homogeneous**. The actual computed F is:

$$F = \frac{t_z}{f^2}\begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix}$$

Multiplying F by scalar $t_z/f^2$ scales both sides of $\mathbf{x}'^T F \mathbf{x} = 0$ equally — zero is still zero. So the scalar is **irrelevant** and dropped. The magnitude and scale of translation do not affect *which* epipolar lines exist, only the *scale* of F — and scale is meaningless for F.

***
## 7. Estimating F: The 8-Point Algorithm
### 7.1 Where Do Matched Points Come From?
Before estimating F, M matched pairs $\{(\mathbf{x}_m, \mathbf{x}_m')\}$ are obtained through a **feature matching pipeline**:

**Stage 1 — Detect keypoints** independently in both images. Keypoints are "interesting" pixels — corners, blobs — that are repeatable across viewpoints. Common detectors: Harris (eigenvalue-based corners), SIFT (scale-space extrema), ORB (FAST corners, very fast).

**Stage 2 — Compute descriptors.** For each keypoint, compute a compact vector encoding local appearance:

| Descriptor | Type | Dimension |
|---|---|---|
| SIFT | Gradient histogram | 128-D float |
| ORB | Binary pixel comparisons | 32-D binary |

Descriptors are invariant to rotation, scale, and lighting — so the same physical scene point produces nearly identical descriptors in both images.

**Stage 3 — Match descriptors** using nearest-neighbour search. Apply **Lowe's ratio test** to filter ambiguous matches:

$$\frac{d_{\text{nearest}}}{d_{\text{2nd nearest}}} < 0.75$$

A match is accepted only if the nearest neighbour is clearly better than the second-nearest.

**Stage 4 — RANSAC outlier rejection.** Even after the ratio test, some matches are wrong (outliers). RANSAC:
1. Randomly sample 8 matches → compute F
2. Count matches satisfying $|\mathbf{x}'^T F \mathbf{x}| < \varepsilon$ — these are **inliers**
3. Repeat many iterations → keep F with the most inliers

The final M pairs are the **clean RANSAC inliers** passed to the 8-point algorithm.
### 7.2 Setting Up the Linear System
Flatten F row-by-row into a 9-vector $\mathbf{f} = (f_1, \ldots, f_9)^T$. Each matched pair expands $\mathbf{x}_m'^T F \mathbf{x}_m = 0$ into:

$$x_m x_m' f_1 + x_m y_m' f_2 + x_m f_3 + y_m x_m' f_4 + y_m y_m' f_5 + y_m f_6 + x_m' f_7 + y_m' f_8 + f_9 = 0$$

This is one linear equation in 9 unknowns — **one row** of a matrix system. Stack M pairs:

$$\underbrace{\begin{bmatrix} x_1x_1' & x_1y_1' & x_1 & y_1x_1' & y_1y_1' & y_1 & x_1' & y_1' & 1 \\ \vdots & & & & & & & & \vdots \\ x_Mx_M' & x_My_M' & x_M & y_Mx_M' & y_My_M' & y_M & x_M' & y_M' & 1 \end{bmatrix}}_{A\;(M \times 9)} \begin{pmatrix}f_1\\\vdots\\f_9\end{pmatrix} = \mathbf{0}$$

**A is M×9 because:** there are 9 unknowns (entries of F → 9 columns) and M point-pair equations (M rows), one per correspondence.
### 7.3 Why 8 Points, Not 9?
F has **7 degrees of freedom**, not 9. The two reductions are:

- **−1 for homogeneous scale:** F is defined up to a non-zero scalar. The constraint $\mathbf{x}'^T F \mathbf{x} = 0$ is unchanged if F → λF. So one DOF is always free — fixed by the unit-norm constraint $\|\mathbf{f}\| = 1$, not by setting any individual entry to 1 (which would fail if that entry is near zero).
- **−1 for the rank-2 constraint:** $\det F = 0$ removes one more DOF. However, this is enforced *after* solving the linear system, not within it — so it does not contribute an equation here.

Result: 9 − 1 (scale) = **8 independent constraints needed → 8 point pairs**.

> **Compare with Homography H:** H has 8 DOF and each point pair gives **2 scalar equations** (because $\mathbf{x}' = H\mathbf{x}$ is a 2D vector equation). So only 4 pairs are needed. For F, $\mathbf{x}'^T F \mathbf{x} = 0$ is a **scalar** equation — only 1 per pair — hence 8 pairs.
### 7.4 Solving the Homogeneous System: Total Least Squares
The trivial solution $\mathbf{f} = \mathbf{0}$ satisfies $A\mathbf{f} = \mathbf{0}$ but is useless. Constrain the norm to exclude it:

$$\min_{\mathbf{f}} \|A\mathbf{f}\|^2 \quad \text{subject to} \quad \|\mathbf{f}\|^2 = 1$$

**Solution: SVD of A.**

$$A = U\Sigma V^T$$

The solution **f** is the **last column of V** — the right singular vector corresponding to the **smallest singular value** of A. This minimises $\|A\mathbf{f}\|^2$ under the unit-norm constraint exactly. Reshape this 9-vector back into a 3×3 matrix to get the raw F.
### 7.5 The Complete 8-Point Algorithm
| Step | Action | Why It's Needed |
|---|---|---|
| **0. Normalize** | Translate each image's points so centroid = origin; scale so avg distance from origin = √2 | Raw pixel coordinates (e.g. 343, 221) create huge numerical range in A — products like $x_mx_m'$ can be ~100,000 while the last entry is 1 — making A ill-conditioned and SVD unreliable |
| **1. Build A** | Construct M×9 matrix from normalized point pairs | Encodes all epipolar constraints linearly |
| **2. SVD of A** | Compute $A = U\Sigma V^T$ | Solves the homogeneous least-squares system |
| **3. Extract F** | Reshape last column of V into 3×3 → F_raw | Minimises residual $\|A\mathbf{f}\|^2$ |
| **4. Enforce rank 2** | SVD of F_raw: zero out smallest singular value → $F = U\,\text{diag}(\sigma_1,\sigma_2,0)\,V^T$ | Raw F is generically rank 3 (noise fills the third singular value); true F must be rank 2 |
| **5. Un-normalize** | $F_{\text{final}} = T'^T F_{\text{rank2}} T$ | Converts F back from normalised to original pixel coordinates |

**Why Step 4 gives the best rank-2 approximation:** By the **Eckart–Young theorem**, the best rank-2 approximation to any matrix (in Frobenius norm) is obtained by zeroing the smallest singular value in its SVD. This is mathematically optimal.
### 7.6 Finding Epipoles from F
Once F is in hand, epipoles fall out of the SVD at no extra cost:

$$F = U\Sigma V^T$$
$$\mathbf{e} = \text{last column of } V \quad (F\mathbf{e} = \mathbf{0} \;\text{— right null vector})$$
$$\mathbf{e}' = \text{last column of } U \quad (F^T\mathbf{e}' = \mathbf{0} \;\text{— left null vector})$$

***
## 8. Full Pipeline
| Stage | Description |
|---|---|
| Two images | Image 1 and image 2 |
| Feature matching pipeline | Detect keypoints, compute descriptors, match with nearest neighbour plus Lowe's test, then apply RANSAC to obtain clean inlier pairs |
| Matched pairs | $\{(\mathbf{x}_m, \mathbf{x}_m')\}$ |
| 8-point algorithm | Normalize coordinates $(T, T')$, build $M \times 9$ matrix $A$, compute SVD, extract $F_{\text{raw}}$, enforce rank 2, then un-normalize |
| Fundamental matrix | $F \in \mathbb{R}^{3\times 3}$, rank 2 |
| Epipolar lines | $\mathbf{l}' = F\mathbf{x}$ in image 2 and $\mathbf{l} = F^T\mathbf{x}'$ in image 1 |
| Epipoles | $\mathbf{e}$ is the right null vector of $F$; $\mathbf{e}'$ is the left null vector of $F$ |

***
## 9. Quick Reference
### Key Equations
| Equation | Meaning |
|---|---|
| $\mathbf{x}'^T F \mathbf{x} = 0$ | Fundamental epipolar constraint |
| $\mathbf{l}' = F\mathbf{x}$ | Epipolar line in image 2 for point **x** in image 1 |
| $\mathbf{l} = F^T\mathbf{x}'$ | Epipolar line in image 1 for point **x′** in image 2 |
| $F = K'^{-T}[\mathbf{t}]_\times R K^{-1}$ | F computed from known camera matrices |
| $F\mathbf{e} = \mathbf{0}$ | **e** is the right null vector of F |
| $F^T\mathbf{e}' = \mathbf{0}$ | **e′** is the left null vector of F |
### Properties of F
- 3×3 matrix, defined **up to scale** (homogeneous)
- **Rank 2**, determinant = 0
- **7 degrees of freedom** (9 entries − 1 scale − 1 rank constraint)
- $F^T$ is the fundamental matrix for the **reversed** camera pair
- Encodes the complete epipolar geometry between the two views
### Special Cases at a Glance
| Camera Motion | F (up to scale) | Epipoles | Epipolar lines |
|---|---|---|---|
| Parallel (side-by-side) | $\begin{bmatrix}0&0&0\\0&0&-1\\0&1&0\end{bmatrix}$ | At infinity $(1,0,0)^T$ | Horizontal scan lines $y = y'$ |
| Forward translation | $\begin{bmatrix}0&-1&0\\1&0&0\\0&0&0\end{bmatrix}$ | At image centre $(0,0,1)^T$ | Radial lines from centre |