# 📚 Image Homographies — Master Notes


## 1. Motivation: Panoramas

A **panorama** is an image with a (near) 360° field of view, created by stitching multiple images from different viewpoints. Wide-angle fish-eye lenses can do this optically but are expensive, bulky, and introduce heavy distortion. Image stitching with homographies is the practical alternative.

**Why not simple translation stitching?** When images are captured from different angles, perspective changes cause misalignment that pure translation cannot fix. A full projective transformation — a **homography** — is required.

---

## 2. Projective Geometry Fundamentals

### Standard vs Projective 2D Space

| Space | Representation | Key Property |
| --- | --- | --- |
| Standard ℝ² | 2 coordinates $(x, y)$ | Euclidean geometry |
| Projective ℙ² | 3 coordinates $(x, y, z)$ | $(\lambda x, \lambda y, \lambda z) \equiv (x, y, z)$ |

### Mappings

**ℝ² → ℙ²** (to homogeneous):
$$(x, y) \rightarrow (x, y, 1)$$

**ℙ² → ℝ²** (back to image coordinates, _dehomogenize_):
$$(x, y, z) \rightarrow \left(\frac{x}{z},\ \frac{y}{z}\right)$$

**Why homogeneous coordinates?** In a pinhole camera, all points $(\lambda x, \lambda y, \lambda)$ on the same ray map to the same image point $(x, y, 1)$. Homogeneous coordinates naturally capture this scale ambiguity and make perspective projection **linear**.

---

## 3. Classification of 2D Transformations

| Transform | Matrix | DOF | Preserves |
| --- | --- | --- | --- |
| Translation | $[I \mid t]_{2\times3}$ | 2 | Orientation, lengths, angles, parallelism |
| Rigid (Euclidean) | $[R \mid t]_{2\times3}$ | 3 | Lengths, angles, parallelism |
| Similarity | $[sR \mid t]_{2\times3}$ | 4 | Angles, parallelism |
| Affine | $[A]_{2\times3}$ | 6 | Parallelism |
| **Projective (Homography)** | $[\tilde{H}]_{3\times3}$ | **8** | **Straight lines only** |

Each row is strictly more general than the one above. Homographies are the most expressive — they only guarantee lines remain lines.

---

## 4. Homography: Definition & Properties

### Mathematical Definition

A homography is a **3×3 matrix H** mapping points between two projective planes sharing the same camera center:
$$\begin{bmatrix} x' \\ y' \\ w' \end{bmatrix} = \begin{bmatrix} h_1 & h_2 & h_3 \\ h_4 & h_5 & h_6 \\ h_7 & h_8 & h_9 \end{bmatrix} \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}$$

Then **dehomogenize**: $(x', y', w') \rightarrow (x'/w',\ y'/w')$

### Degrees of Freedom

H has 9 entries but only **8 DOF** because $\alpha H$ and $H$ describe the **same geometric transformation** (scale ambiguity). Therefore:
$$\text{Minimum correspondences needed} = \frac{8\ \text{DOF}}{2\ \text{constraints/point}} = 4\ \text{points}$$

### Properties

- ✅ Straight lines map to straight lines
- ✅ Compositions of homographies are homographies — if $H_1$ maps A→B and $H_2$ maps B→C, then $H_2 H_1$ maps A→C and is still a valid homography
- ❌ Parallel lines do **not** necessarily remain parallel
- ❌ Angles, ratios, and origin are **not** preserved

### When Can We Use Homographies?

Three valid conditions — all require the same center of projection:

1. **Planar scene** — all world points lie on a single plane (e.g., walls, documents)
2. **Approximately planar** — scene is very far away; even large absolute depth differences become negligible _relative_ to the camera distance, so the scene behaves as if flat (e.g., distant landscapes)
3. **Camera rotation only** — no translation, only rotation about the camera center (e.g., tripod panoramas)

---

## 5. Applying a Homography

1. **Convert to homogeneous:** $p = [x,\ y,\ 1]^T$
2. **Multiply:** $p' = H \cdot p = [x',\ y',\ w']^T$
3. **Dehomogenize:** $\rightarrow (x'/w',\ y'/w')$

---

## 6. Direct Linear Transform (DLT)

### Goal

Given matched correspondences $\{p_i \leftrightarrow p'_i\}$, find H such that $p'_i = H \cdot p_i$.

### Derivation

For a single correspondence $(x, y) \rightarrow (x', y')$, expanding $p' = \alpha H p$ and eliminating the scale factor $\alpha$ (by dividing row 1 and 2 by row 3):

$$x' = \frac{h_1 x + h_2 y + h_3}{h_7 x + h_8 y + h_9}, \qquad y' = \frac{h_4 x + h_5 y + h_6}{h_7 x + h_8 y + h_9}$$

Cross-multiplying and rearranging into linear form:

$$-xh_1 - yh_2 - h_3 + x'xh_7 + x'yh_8 + x'h_9 = 0$$
$$-xh_4 - yh_5 - h_6 + y'xh_7 + y'yh_8 + y'h_9 = 0$$

### Constraint Matrix

Each correspondence produces a **2×9 matrix** $A_i$:
$$A_i = \begin{bmatrix} -x & -y & -1 & 0 & 0 & 0 & x'x & x'y & x' \\ 0 & 0 & 0 & -x & -y & -1 & y'x & y'y & y' \end{bmatrix}$$

Stack $n$ correspondences into a **$2n \times 9$** matrix:
$$A = \begin{bmatrix} A_1 \\ A_2 \\ \vdots \\ A_n \end{bmatrix} \implies Ah = 0, \quad h = [h_1,\ h_2,\ \ldots,\ h_9]^T$$

**With minimum 4 points:** A is **8×9** — exactly 1D null space, giving a unique solution up to scale.
**With $n > 4$ noisy points:** Overdetermined — use least squares.

### Fitting Homographies — Two Regimes

Homography has $9$ parameters, but scale is unobservable ⇒ only $8$ DOF ⇒ minimum $4$ correspondences. Fix the gauge with $\|h\|^2 = 1$ and the fitting problem becomes:

| Regime | Objective |
|---|---|
| **Noiseless** ($n = 4$, exact) | $Ah = 0 \quad \text{s.t.} \quad \|h\|^2 = 1$ |
| **Noisy** ($n > 4$, overdetermined) | $\displaystyle \min_h \|Ah\|^2 \quad \text{s.t.} \quad \|h\|^2 = 1$ |

Both are solved by the **last column of $V$** in $A = U \Sigma V^T$ (proof below).

### Why Homogeneous Least Squares?

Without a constraint, $h = 0$ trivially satisfies $Ah = 0$ — completely useless. Since $\alpha H \equiv H$ geometrically, we fix the scale with $\|h\| = 1$:

$$\min_h \|Ah\|^2 \quad \text{subject to} \quad \|h\|^2 = 1$$

Equivalently (Rayleigh Quotient):
$$\min_h \frac{\|Ah\|^2}{\|h\|^2}$$

This is minimized by the **eigenvector of $A^TA$ corresponding to its smallest eigenvalue**.

The $\|h\| = 1$ constraint doesn't add an equation to A — it selects **one representative** from the infinite family $\{\alpha h^*\}$ by intersecting the null space line with the unit hypersphere. Since V's columns are unit vectors by definition, SVD satisfies this constraint automatically.

### SVD Solution — Proof

SVD: $A = U \Sigma V^T$, with $U, V$ orthogonal and $\Sigma = \mathrm{diag}(\sigma_1 \geq \cdots \geq \sigma_n \geq 0)$. Substitute $z = V^T h$:

$$\|Ah\|^2 = \|U \Sigma V^T h\|^2 = \|\Sigma z\|^2 = \sum_{i=1}^{n} \sigma_i^2 z_i^2,$$

$$\|h\|^2 = \|V z\|^2 = \|z\|^2 = 1 \quad (\text{V orthogonal}).$$

Minimise $\sum_i \sigma_i^2 z_i^2$ subject to $\sum_i z_i^2 = 1$ ⇒ concentrate all weight on the smallest $\sigma_i$:

$$z^* = e_n = [0, \ldots, 0, 1]^T \implies h^* = V z^* = v_n \ \ (\text{last column of } V).$$

**Uniqueness:** holds iff $\sigma_{n-1} > \sigma_n$. 

Noiseless 4-point case: $\sigma_9 = 0$ (1D null space). 

Noisy case: $\sigma_9 \approx 0$, $v_9$ is the best approximation.

### What V Encodes (The Four Fundamental Subspaces)

For $A = U\Sigma V^T$ with rank $r$:

| Columns of V | Subspace |
| --- | --- |
| First $r$ columns | **Row space** of A |
| Last $n-r$ columns | **Null space** of A |

| Columns of U | Subspace |
| --- | --- |
| First $r$ columns | **Column space** of A |
| Last $m-r$ columns | **Left null space** of A |

### Recovering H from h

The 9-element solution vector h is simply **reshaped** row-by-row back into the 3×3 matrix:

$$h = [h_1, h_2, \ldots, h_9]^T \implies H = \begin{bmatrix} h_1 & h_2 & h_3 \\ h_4 & h_5 & h_6 \\ h_7 & h_8 & h_9 \end{bmatrix}$$

Optional normalization (standard convention):
$$H_\text{final} = \frac{1}{h_9}\ H$$

### Complete DLT Algorithm

**Input:** $\{x_i \leftrightarrow x'_i\}$, minimum 4 non-collinear point pairs

1. Build **2×9** matrix $A_i$ for each correspondence
2. Stack into **$2n\times9$** matrix A
3. Compute SVD: $A = U\Sigma V^T$
4. $h$ = last column of V (smallest singular value)
5. Reshape $h \in \mathbb{R}^9 \rightarrow H \in \mathbb{R}^{3\times3}$

---

## 7. Effect of Wrong Correspondences on DLT

With exactly 4 points, DLT finds H that **perfectly fits all 4 given correspondences** — including the wrong one. There is no averaging or resistance. One bad correspondence completely changes H.

**The insidious part:** Training error stays zero on the 3 correct points. The wrong H is only detectable by testing on **unseen points** outside the training set. This is why DLT alone is fragile — and why RANSAC is needed in practice.

Empirically, a single correspondence shifted by 80px (out of 4 total) causes ~21px mean error on unseen test points. Even 10px corruption propagates to ~2px error everywhere.

---

## 8. Feature Matching: How Correspondences Are Obtained

Correspondences are produced **automatically** in three stages:

### Stage 1: Feature Detection

Detect distinctive, repeatable **keypoints** (e.g., corners) using the **Harris Corner Detector** — points where intensity changes sharply in multiple directions.

### Stage 2: Feature Description

Compute a 128-dimensional **descriptor** vector (SIFT) around each keypoint, invariant to rotation, scale, and mild lighting changes.

### Stage 3: Feature Matching

Pair keypoints from both images whose descriptors are most similar (nearest-neighbour search):
$$j = \arg\min_j \|d_i - d'_j\|_2$$

**Lowe's ratio test** filters ambiguous matches:
$$\frac{\|d_i - d'_\text{best}\|}{\|d_i - d'_\text{second best}\|} < 0.8$$

**Output:** A large list of **putative correspondences** $\{p_i \leftrightarrow p'_i\}$ — a mix of inliers (correct) and outliers (wrong pairings due to repetitive textures or ambiguous patches).

---

## 9. RANSAC: Robust Estimation Under Outliers

### Why It's Needed

Feature matchers produce incorrect pairs a significant fraction of the time. Standard DLT (least squares) fits the "average" transform — corrupted by outliers. **RANSAC selects which correspondences to trust, then DLT computes the actual H.**

### How Inliers vs Outliers Are Determined

After computing a candidate H, apply it to every correspondence and compute **reprojection error**:

$$\text{error}_i = \|p'_i - H \cdot p_i\|_2$$

A point is an **inlier** if $\text{error}_i < \tau$. Threshold $\tau$ is set from expected noise level (typically $\tau \approx 3$ pixels for standard feature detectors, derived from Gaussian noise model: $\tau = \sigma\sqrt{\chi^2_{2,0.95}} \approx 3\sigma$).

### What "All Correspondences" Means

The feature matcher produces (say) 500 putative pairs **before** RANSAC starts. In each RANSAC iteration:

- **4 are randomly sampled** to compute a candidate H
- That H is **tested against all 500** to count total inliers

The sampled 4 define the hypothesis. The full 500 **validate** it. This is why a spurious H accidentally fitting 4 wrong points almost always fails — it will score very few inliers across the full 500.

### RANSAC Algorithm

**Repeat for N iterations:**

1. Randomly sample 4 correspondences
2. Compute candidate H via DLT
3. Test H on **all** correspondences; count inliers $(\text{error} < \tau)$
4. Keep H if it has the highest inlier count so far

**After all iterations:** Recompute final H using DLT on **all inliers** of the best model.

### How Many Iterations?

- $w$ — probability a randomly chosen point is an inlier.
- $k$ — points per sample.
- $p$ — desired success probability.
- $N$ — number of RANSAC iterations.

$$P(\text{one good trial}) = w^k$$
$$P(\text{success in N trials}) = 1 - (1-w^k)^N \geq p$$
$$N \geq \frac{\log(1-p)}{\log(1-w^k)}$$

**N is not the expected number of trials** — it is a worst-case probabilistic guarantee. The expected number of trials until first success is $1/w^k$ (geometric distribution), which is far lower. If success is found early, you can stop.

### How w Is Determined

- **Before RANSAC starts:** User provides a rough estimate based on expected match quality (e.g., 0.5–0.7 for typical SIFT matching)
- **During RANSAC:** Updated adaptively after each trial as the observed inlier ratio of the current best model:
$$w \leftarrow \frac{\text{current best inlier count}}{\text{total correspondences}}$$
Then N is recomputed dynamically — if a high-inlier-ratio model is found early, N shrinks and RANSAC terminates early.

### Example (for $p = 0.99$)

| $k$ | $w=90\%$ | $w=80\%$ | $w=70\%$ | $w=60\%$ | $w=50\%$ |
| --- | --- | --- | --- | --- | --- |
| **4** | **5** | **9** | **17** | **34** | **72** |

*Reading the table:* with $k=4$ points per sample and an inlier rate $w = 0.90$, only $N = 5$ random trials suffice to be $99\%$ sure ($p = 0.99$) that at least one trial drew an all-inlier sample. As $w$ drops, $N$ grows steeply — at $w = 0.50$ the same guarantee needs $72$ trials.

---

## 10. Image Stitching: After H Is Found

### Step 1: Canvas Sizing

Project the **4 corners** of Image 2 through H, then compute the bounding box covering both images. This determines the output canvas size. Black border regions (visible in view synthesis) are output pixels that fall outside both input images.

### Step 2: Inverse Warping

**Forward mapping** (applying H to every pixel of Image 2) causes holes due to rounding. The correct approach is **inverse mapping**:

For every pixel $(x', y')$ in the output canvas, find its source in Image 2:

$$\begin{bmatrix} x \\ y \\ w \end{bmatrix} = H^{-1} \begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix} \implies \text{source} = \left(\frac{x}{w},\ \frac{y}{w}\right)$$

Sample color using **bilinear interpolation** (source is usually non-integer). This guarantees every output pixel is filled — no holes.

Image 1 (the reference) is placed directly on the canvas — no warping needed.

### Step 3: Blending

The overlap region between the two warped images has a hard seam due to exposure differences. Two strategies:

**Alpha (feathering) blending:**
$$\text{output}(x) = \alpha(x) \cdot I_1(x) + (1-\alpha(x)) \cdot I_2(x)$$
where $\alpha$ smoothly transitions 1→0 across the overlap.

**Multi-band blending:** Blends low frequencies (smooth gradients) over a wide region and high frequencies (sharp edges) only near the seam — produces cleaner results.

### Full Pipeline

```
Image 1 + Image 2
       ↓
Harris/SIFT Detection & Description on both images
       ↓
Feature Matching → 500 putative correspondences {pᵢ ↔ p'ᵢ}
       ↓
RANSAC (N iterations):
   ├─ Sample 4 random correspondences
   ├─ Compute H via DLT (SVD)
   ├─ Count inliers across all 500
   └─ Keep best H
       ↓
Recompute H via DLT on all inliers of best model
       ↓
Compute canvas size (project Image 2 corners through H)
       ↓
Inverse warp Image 2 into canvas + place Image 1
       ↓
Blend overlapping region
       ↓
Panoramic output ✅
```

---

## 11. Quick Reference

| Concept | Key Point |
| --- | --- |
| **Homography** | 3×3 matrix; maps between planes sharing same camera center |
| **DOF** | 8 (scale ambiguity); minimum 4 non-collinear point pairs |
| **Matrix A** | $2n \times 9$; 2 rows per correspondence, 9 columns for 9 unknowns |
| **With 4 points** | A is 8×9 — exactly 1D null space → unique solution up to scale |
| **SVD solution** | Last column of V = null space of A (or closest direction to it) |
| **V encodes** | Row space (first $r$ cols) + Null space (last cols) of A |
| **Scale ambiguity fix** | $\|h\|=1$ constraint; automatically satisfied since V has unit columns |
| **DLT** | Solve $\min\|Ah\|^2$ s.t. $\|h\|=1$ via SVD — gives exact H from clean data |
| **RANSAC** | Robust wrapper for DLT; handles outliers via random sampling |
| **Inlier threshold** | $\tau \approx 3$ pixels based on Gaussian noise model |
| **N (RANSAC iters)** | Worst-case budget for $p$ confidence, not the expected stopping point |
| **Inverse warping** | Apply $H^{-1}$ per output pixel — avoids holes from forward mapping |
| **Valid use cases** | Planar scene, approximately planar (distant), camera rotation only |
