# Corner Detection — Comprehensive Notes
*EE5178: Modern Computer Vision*

***

## 1. Motivation

Vision tasks like **stereo reconstruction**, **motion estimation**, and **image stitching** require finding **corresponding features** across two or more views. The approach is to match small **image patches of fixed size** between frames.

- **Good patch**: Distinctive texture (e.g., a corner) — only *one* patch in the second image looks like it
- **Bad patch**: Uniform region or edge — *many* patches look similar, making the match ambiguous

***

## 2. What is a Corner?

A **corner** is a point where there is a **junction of two or more contours (edges)** — equivalently, a point whose local neighbourhood exhibits **large intensity variation in all directions**. Shifting a small window at a corner in *any* direction produces a large change in appearance.

![Three cases of window sliding](Images/Sliding_Window.png)

Three types of local regions when sliding a window:

| Region | Window Response | Gradient Scatter Shape |
|---|---|---|
| **Flat** | No change in any direction | Tight dot at origin |
| **Edge** | Change only perpendicular to edge | Elongated ellipse along one axis |
| **Corner** | Large change in **all** directions | Broad circular blob |

Corners are preferred as features because they are **stable across viewpoint changes**, **uniquely localizable**, and **good for matching**.

***

## 3. The SSD Error Function

The core measurement: how much does intensity change when a window is shifted by $(u, v)$?

$$E(u,v) = \sum_{x,y} w(x,y)\,[I(x+u,\,y+v) - I(x,y)]^2$$

- $w(x,y)$: weighting function — Gaussian (preferred) or rectangular
- Goal: find patches where **$E(u,v)$ is large for all shift directions**
- Flat region: $E \approx 0$ for all shifts; Corner: $E$ is large

***

## 4. Taylor Series Approximation

The full 2D Taylor expansion of a shifted function is:

$$
f(x+u, y+v) = f(x,y) + uf_x + vf_y + \frac{1}{2!}[u^2f_{xx} + uvf_{xy} + v^2f_{yy}] + \cdots
$$

For **small shifts**, all terms beyond first order are dropped (**small motion assumption**):

$$
I(x+u, y+v) \approx I(x,y) + uI_x + vI_y
$$

Substituting into $E(u,v)$ — the $I(x,y)$ terms cancel:

$$
E(u,v) \approx \sum w(x,y)\,[uI_x + vI_y]^2 = \sum u^2I_x^2 + 2uvI_xI_y + v^2I_y^2
$$

***

## 5. The Structure Tensor M

Rewriting as a **bilinear (matrix) form**:

$$
E(u,v) \cong \begin{bmatrix}u & v\end{bmatrix} M \begin{bmatrix}u \\ v\end{bmatrix}
$$

where $M$ is the **2×2 Structure Tensor** (Second Moment Matrix):

$$
M = \sum_{x,y} w(x,y) \begin{bmatrix} I_x^2 & I_xI_y \\ I_xI_y & I_y^2 \end{bmatrix}
$$

The three entries — $S_{x^2}, S_{xy}, S_{y^2}$ — are computed by:
1. **Hadamard products**: $I_x^2 = I_x \circ I_x$, $I_y^2 = I_y \circ I_y$, $I_{xy} = I_x \circ I_y$ (elementwise squaring/multiplication — **not** matrix multiplication)
2. **Gaussian-weighted summation**: $S_{x^2} = \sum G_I \circ I_x^2$, and so on

### Eigenvalue Interpretation

The eigenvalues $\lambda_1, \lambda_2$ of $M$ describe the **principal axes of the gradient scatter ellipse**:

![Classification via eigenvalues](Images/classification_eigenvalues.png)

| $\lambda_1, \lambda_2$ | Scatter Shape | Region |
|---|---|---|
| Both ≈ 0 | Tiny dot at origin | **Flat** |
| $\lambda_1 \gg 0,\ \lambda_2 \approx 0$ | Elongated ellipse | **Edge** |
| Both large | Broad circular blob | **Corner** ✅ |

***

## 6. Corner Response R

To avoid expensive eigenvalue computation, Harris uses:

$$
\boxed{R = \det(M) - k\,(\text{trace}\,M)^2}
$$

where:
- $\det(M) = \lambda_1\lambda_2 = S_{x^2} \cdot S_{y^2} - S_{xy}^2$
- $\text{trace}(M) = \lambda_1 + \lambda_2 = S_{x^2} + S_{y^2}$
- $k = 0.04\text{–}0.06$ (empirically determined)

| R value | Verdict |
|---|---|
| $R \gg 0$ | **Corner** — both $\lambda$ large, $\det(M)$ dominates |
| $R < 0$ (large negative) | **Edge** — one $\lambda \approx 0$, so $\det(M) = 0$ |
| $|R| \approx 0$ | **Flat** — both $\lambda$ small |

The key insight: $\det(M) = \lambda_1\lambda_2$ is **zero whenever either eigenvalue is zero** — this kills the edge case — and **large only when both are substantial** — the corner case.

***

## 7. Full Algorithm Pipeline

### Step 1 — Pre-smooth with $\sigma_D$
$$I_x = G_{\sigma_D}^x * I, \quad I_y = G_{\sigma_D}^y * I$$
Apply a Gaussian derivative filter to suppress pixel-level noise **before** differentiation. A larger $\sigma_D$ gives coarser but more stable gradients.

### Step 2 — Compute Sobel gradients

Using 3×3 Sobel kernels on the $\sigma_D$-smoothed image:
$$
K_x = \begin{bmatrix}-1&0&1\\-2&0&2\\-1&0&1\end{bmatrix}, \quad K_y = \begin{bmatrix}-1&-2&-1\\0&0&0\\1&2&1\end{bmatrix}
$$
$K_x$ fires on **vertical edges** (horizontal intensity change); $K_y$ fires on **horizontal edges**.

### Step 3 — Gradient products (Hadamard)
$$
I_{x^2} = I_x \circ I_x, \quad I_{y^2} = I_y \circ I_y, \quad I_{xy} = I_x \circ I_y
$$
Pure elementwise operations — cheap $O(mn)$ vs $O(n^3)$ matrix multiply.

### Step 4 — Smooth with $\sigma_I$
$$
S_{x^2} = G_{\sigma_I} * I_{x^2}, \quad S_{y^2} = G_{\sigma_I} * I_{y^2}, \quad S_{xy} = G_{\sigma_I} * I_{xy}
$$
The Gaussian window **aggregates neighbourhood evidence** into the structure tensor. The centre pixel gets the highest weight (e.g., 0.2042 for $\sigma=1$); corner pixels get the least (0.0751).

### Step 5 — Build structure tensor M
$$
M = \begin{bmatrix}S_{x^2} & S_{xy}\\ S_{xy} & S_{y^2}\end{bmatrix}
$$

### Step 6 — Compute R, threshold, NMS
$$
R = \det(M) - k\,(\text{trace}\,M)^2
$$
Threshold on $R$, then apply **Non-Maximum Suppression** — keep only local maxima within a neighbourhood to thin detections to one point per corner.

### Why Two Different Smoothings?

| Parameter | Symbol | Typical $\sigma$ | Purpose |
|---|---|---|---|
| Differentiation scale | $\sigma_D$ | **smaller** (~0.5–1.0) | Pre-smooth image to suppress noise *before* differentiation |
| Integration scale | $\sigma_I$ | **larger** (~1.5–2.0) | Gaussian window aggregates gradient energy over neighbourhood |

They serve completely different purposes — one prepares the image; the other builds the neighbourhood summary. Standard relationship: $\sigma_I = \gamma\,\sigma_D$ with $\gamma \approx 1.5\text{–}2$.

***

## 8. Worked Numerical Summary

All three cases use a **7×7 patch**, $k=0.04$:

| | Flat | Edge | Corner |
|---|---|---|---|
| Patch | Uniform ~101 | Dark left / bright right | Dark top-left / bright bottom-right |
| After $\sigma_D$ | Uniform | Smooth ramp in x only | Smooth ramp in **both** x and y |
| $I_x$ | ~0 | **Large** (~552) | **Large** (~376) |
| $I_y$ | ~0 | **0** | **Large** (~376) |
| $S_{x^2}$ | 0.18 | 232,722 | 110,691 |
| $S_{y^2}$ | 0.40 | **0.00** | 110,691 |
| $S_{xy}$ | 0.07 | 0.00 | 73,564 |
| $\det(M)$ | 0.07 | **0.00** | **6.84×10⁹** |
| $\text{tr}(M)$ | 0.58 | 232,722 | 221,382 |
| $R$ | **+0.05** | **−2.17×10⁹** | **+4.88×10⁹** |
| Verdict | ⬜ Flat | ❌ Edge | ✅ Corner |

Key observation for the Edge case: $\det(M) = S_{x^2} \cdot S_{y^2} - S_{xy}^2 = 232722 \times 0 - 0 = 0$, so $R = -k\,\text{tr}(M)^2$, a large negative.

***

## 9. Invariance Properties

| Transform | Invariant? | Reason |
|---|---|---|
| **Translation** | ✅ Yes | Only local gradient properties used |
| **Rotation** | ✅ Yes | Ellipse rotates but eigenvalues $\lambda_1, \lambda_2$ stay the same; $R$ depends only on eigenvalues |
| **Additive intensity** $I \to I + b$ | ✅ Yes | Derivatives cancel the constant |
| **Multiplicative intensity** $I \to aI$ | ⚠️ Partial | $R$ scales by $a^2$; threshold must be rescaled |
| **Scale (zoom)** | ❌ No | Fixed window can't adapt to image scale |
| **Affine transform** | ❌ No | Stretching changes eigenvalue ratios |

### Rotation Invariance — Key Insight
When a patch rotates, the **eigenvectors** of $M$ (directions of max/min change) rotate with it, but the **eigenvalues** (magnitudes) remain unchanged. Since $R$ is computed purely from eigenvalues ($\det M$ and $\text{tr} M$), it is rotation-invariant.

### Affine Transform — What It Is
An affine transform = **Translation + Rotation + Scaling + Shear** — formally $f(x) = Ax + b$. It preserves straight lines and parallelism but **not** angles or distances. It is a **special case of projective (homography) transform** — the projective transform adds perspective warp (8 DoF) where parallel lines can converge. Affine is projective with the last row of the $3\times3$ matrix fixed as $[0\ 0\ 1]$.

***

## 10. Scale Invariance Failure → Motivation for SIFT

Harris is **not scale-invariant**. The same physical corner viewed at a different zoom level may be classified as an edge — the fixed window sees different amounts of structure.

This directly motivates **scale-invariant detectors**:
- **LoG / DoG blob detection** — finds characteristic scale by searching extrema across a scale-space pyramid
- **SIFT** — detects corners and their characteristic scale automatically, inheriting Harris's rotation invariance while adding scale invariance

***

## 11. Gradient Distribution & Ellipse Fitting

Each pixel in the window contributes a point $(I_x, I_y)$ to a 2D scatter plot. The **shape of this cloud** is characterized by fitting a principal component ellipse — its axes are the eigenvectors of $M$, and its radii are $\sqrt{\lambda_1}, \sqrt{\lambda_2}$:

![Gradient distribution for different regions](Images/gradient_distribution_corners.png)

- **Flat**: Tiny circular cluster near origin → both $\lambda$ small
- **Edge**: Elongated ellipse along one axis → one $\lambda$ large, one small
- **Corner**: Large circular/isotropic blob → both $\lambda$ large

This geometric picture is the intuitive bridge between the gradient images and the algebraic corner response $R$.
