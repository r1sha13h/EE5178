# 📘 Stereo Reconstruction — Comprehensive Notes

***

## 1. Motivation & Overview

**Stereo reconstruction** is the process of recovering 3D shape from two (or more) 2D images taken from known camera viewpoints. It is the computational equivalent of human binocular vision — two slightly offset views of the same scene allow depth to be inferred.

**The two-step pipeline:**
1. **Correspondence** — for each point in image 1, find the matching point in image 2 *(search problem)*
2. **Triangulation** — given the matched pair, compute the 3D position *(estimation problem)*

***

## 2. Epipolar Geometry

### The Correspondence Problem

A naïve search for a match would scan the entire 2D image — expensive and ambiguous. The **epipolar constraint** reduces this to a **1D search** along a line.

- A 3D point \(\mathbf{X}\) projects to point \(\mathbf{x}\) in image 1
- The back-projected ray from \(\mathbf{x}\) through camera 1 intersects the image plane of camera 2 along a line — the **epipolar line**
- The true match \(\mathbf{x}'\) **must lie on this epipolar line**

### Parallel vs. Converging Cameras

| Camera Type | Epipolar Lines | Consequence |
|------------|---------------|-------------|
| **Parallel** (co-planar image planes) | Horizontal scan lines — perfectly parallel | Correspondence = 1D horizontal search |
| **Converging** (toed-in) | Fan out from epipole \(e\), tilted and non-parallel | Must apply **rectification** first |

### Rectification

Rectification converts converging camera geometry into equivalent parallel geometry by applying a **2D homography** (projective transformation, \(3\times3\) matrix \(H\)) to each image independently. Two different homographies \(H_1\), \(H_2\) are computed such that after warping, corresponding epipolar lines lie on the **same row** in both images. All subsequent stereo algorithms that assume parallel cameras can then be applied directly.

***

## 3. Dense Stereo Correspondence

Dense stereo computes a depth estimate at **every pixel**.

### Cross-Correlation Matching

**Algorithm:**
For each pixel \(i\) in the left image:
1. Extract neighbourhood patch \(W_1(i)\) — a small window of pixel intensities (e.g., 5×5 or 11×11) centred at \(i\)
2. Slide along the epipolar line in the right image
3. At each candidate position, compute cross-correlation with patch \(W_2(i + d)\)
4. Pick the position with **highest cross-correlation** as the match

**Cross-correlation formula** (mean-subtracted, normalized):
\[\text{NCC} = \frac{(\mathbf{a} - \langle\mathbf{a}\rangle) \cdot (\mathbf{b} - \langle\mathbf{b}\rangle)}{|\mathbf{a} - \langle\mathbf{a}\rangle|\ |\mathbf{b} - \langle\mathbf{b}\rangle|}\]
This is invariant to affine intensity changes \(I \to \alpha I + \beta\).

**Parameters:**
- **Window size** — larger windows are more stable but blur depth boundaries; smaller windows give sharper edges but noisier results
- **Search disparity range** — how far along the epipolar line to search; must cover the expected near/far depth range

***

## 4. Limitations of the Similarity Constraint

Cross-correlation assumes corresponding patches look the same in both images. This breaks in four cases:

### 4.1 Textureless Surfaces
*(e.g., Abraham Lincoln's coat, blank walls)*

A textureless patch has no distinctive intensity variation — every candidate along the epipolar line scores equally. The correlation curve is **flat with no peak** → algorithm picks a random match.

### 4.2 Occlusions
Some 3D points visible in camera 1 are **blocked** by a foreground object in camera 2. These points have **no valid correspondence** — but the algorithm is forced to assign one anyway.

### 4.3 Repetitive Patterns
*(e.g., iron fence, tiled floor)*

Multiple positions along the epipolar line produce equally high correlation scores → **multiple peaks** → ambiguous, wrong disparity.

### 4.4 Non-Lambertian Surfaces & Specularities
*(e.g., glossy photo frame, metallic objects)*

A **Lambertian surface** reflects light equally in all directions — it looks the same from any viewpoint. A **non-Lambertian surface** has **view-dependent appearance** — specular highlights (bright glints) that appear at one camera may be completely absent in the other. The patch at the true correspondence looks **entirely different** in both images → cross-correlation fails completely, even the correct match has a low score.

### 4.5 Foreshortening Effects
On surfaces **slanted** relative to the cameras, the same physical patch appears geometrically compressed in one camera vs. the other (one is a stretched version). Cross-correlation only searches for **horizontal shifts** — it cannot handle geometric warping. There is therefore **no meaningful correct match** from cross-correlation's perspective on slanted surfaces.

**Fix:** Affine window adaptation / adaptive support windows — warp the search window to account for estimated surface slant.

***

## 5. Additional Constraints for Disambiguation

Beyond appearance, geometric priors help resolve ambiguity:

- **Uniqueness:** Each pixel in image 1 maps to at most one pixel in image 2
- **Ordering:** If point A is to the left of B in image 1, A's match is also to the left of B's match in image 2 (except at occlusion boundaries)
- **Smoothness of disparity field:** Neighbouring pixels usually have similar depths → the disparity map should be spatially smooth

***

## 6. Disparity and Depth

### The Geometry

For a **parallel camera** setup with:
- Baseline \(b\) = distance between camera centres \(O\) and \(O'\)
- Focal length \(f\)
- 3D point at depth \(Z\)
- Image coordinates \(x\) (left) and \(x'\) (right)

By similar triangles from the left camera:
\[\frac{X}{Z} = \frac{x}{f}\]

From the right camera:
\[\frac{b - X}{Z} = \frac{x'}{f}\]

Subtracting:
\[\boxed{d = x - x' = \frac{bf}{Z}}\]

**Disparity \(d\) is inversely proportional to depth \(Z\).** Inverting: \(Z = \frac{bf}{d}\)

- **Close objects** → large disparity → **bright in disparity map**
- **Far objects** → small disparity → **dark in disparity map**

***

## 7. Stereo Matching as Energy Minimization

The greedy window-based search is locally optimal but globally poor. Reformulating as a **global optimization** problem gives dramatically better results.

### The Energy Function

\[E = \alpha \, E_{\text{data}}(I_1, I_2, D) + \beta \, E_{\text{smooth}}(D)\]

**Data term** — photometric consistency:
\[E_{\text{data}} = \sum_i \bigl(W_1(i) - W_2(i + D(i))\bigr)^2\]
Penalizes large differences between matched patches.

**Smoothness term** — spatial regularization:
\[E_{\text{smooth}} = \sum_{\text{neighbors } i,j} \rho\bigl(D(i) - D(j)\bigr)\]
Penalizes large differences in disparity between adjacent pixels. \(\rho\) is typically a **robust function** (e.g., truncated quadratic) — it penalizes small jumps quadratically but caps the penalty at real depth edges so genuine boundaries are not over-smoothed.

### The \(\alpha/\beta\) Tradeoff

- Large \(\alpha\) → trust image data → noisier depth map
- Large \(\beta\) → enforce smoothness → over-blurred, loses real edges
- Balanced → sharp boundaries + smooth interiors

### Graph Cuts

This energy function is a **Markov Random Field (MRF)**. Finding its global minimum is generally NP-hard but can be solved efficiently for the above form using **graph cuts** (Boykov, Veksler, Zabih, PAMI 2001):

1. Build a graph where each pixel is a node
2. Edges encode data costs and smoothness costs
3. Find the **minimum cut** — this directly gives the optimal disparity assignment

**Result:** Dramatically better depth maps than window-based search, with clean object boundaries and smooth regions. Remaining errors occur at very thin structures and depth discontinuities where the smoothness term competes with the true sharp boundary.

***

## 8. Triangulation

Triangulation recovers the 3D point \(\mathbf{X}\) given matched image points \(\mathbf{x}\), \(\mathbf{x}'\) and known camera matrices \(\mathbf{P}\), \(\mathbf{P}'\).

Due to noise, the two back-projected rays are **skew lines** — they don't intersect. Three methods exist to estimate the best \(\mathbf{X}\):

***

### Method 1 — Midpoint Method
Average the points of closest approach on the two skew rays. Simple but not statistically optimal.

***

### Method 2 — Linear / SVD Method

#### The Projection Equation in Homogeneous Coordinates
\[\mathbf{x} = \alpha \mathbf{P}\mathbf{X}\]
The scale factor \(\alpha\) prevents direct solving. We eliminate it using the **cross product trick**: since \(\mathbf{x}\) and \(\mathbf{P}\mathbf{X}\) point in the same direction,
\[\mathbf{x} \times \mathbf{P}\mathbf{X} = \mathbf{0}\]

#### Expanding with Camera Rows
Writing \(\mathbf{P}\) row-by-row as \(\mathbf{p}_1^\top, \mathbf{p}_2^\top, \mathbf{p}_3^\top\):

\[\begin{bmatrix} y\mathbf{p}_3^\top - \mathbf{p}_2^\top \\ \mathbf{p}_1^\top - x\mathbf{p}_3^\top \\ x\mathbf{p}_2^\top - y\mathbf{p}_1^\top \end{bmatrix}\mathbf{X} = \mathbf{0}\]

The third row is a linear combination of the first two (\(x \times\) row 1 \(+ y \times\) row 2) — discard it. Each camera contributes **2 independent equations**.

#### Full System from Both Cameras

\[\underbrace{\begin{bmatrix} y\mathbf{p}_3^\top - \mathbf{p}_2^\top \\ \mathbf{p}_1^\top - x\mathbf{p}_3^\top \\ y'\mathbf{p}_3'^\top - \mathbf{p}_2'^\top \\ \mathbf{p}_1'^\top - x'\mathbf{p}_3'^\top \end{bmatrix}}_{\mathbf{A} \ (4\times4)} \mathbf{X} = \mathbf{0}\]

#### Solving: SVD
Decompose \(\mathbf{A} = \mathbf{U\Sigma V}^\top\). The solution is the **last column of \(\mathbf{V}\)** — the right singular vector corresponding to the smallest singular value. This minimizes \(\|\mathbf{AX}\|^2\) subject to \(\|\mathbf{X}\|=1\).

#### Backprojection (How to Compute the Ray)
Two points define the ray from camera \(\mathbf{P}\) through image point \(\mathbf{x}\):
1. Camera centre \(C\) — null space of \(\mathbf{P}\): \(\mathbf{PC} = \mathbf{0}\)
2. \(\mathbf{P}^+\mathbf{x}\) — pseudo-inverse of \(\mathbf{P}\) applied to \(\mathbf{x}\); projects back to \(\mathbf{x}\) by definition

***

### Method 3 — Geometric / MLE (Optimal Method)

#### The Idea
Find \(\hat{\mathbf{X}}\) that exactly satisfies camera geometry:
\[\hat{\mathbf{x}} = \mathbf{P}\hat{\mathbf{X}}, \qquad \hat{\mathbf{x}}' = \mathbf{P}'\hat{\mathbf{X}}\]
minimizing the **reprojection error** — the pixel-space distance between observations and projections:

\[\min_{\hat{\mathbf{X}}} \ \mathcal{C}(\mathbf{x}, \mathbf{x}') = d(\mathbf{x}, \hat{\mathbf{x}})^2 + d(\mathbf{x}', \hat{\mathbf{x}}')^2\]

#### Statistical Justification
If measurement noise is Gaussian \(\sim \mathcal{N}(0, \sigma^2)\), then:
\[p(\mathbf{x} \mid \hat{\mathbf{x}}) \propto \exp\!\left(-\frac{d(\mathbf{x}, \hat{\mathbf{x}})^2}{2\sigma^2}\right)\]
Maximizing likelihood = minimizing reprojection error → **this is the Maximum Likelihood Estimate (MLE)** of \(\mathbf{X}\). This is why reprojection error is the standard loss function throughout computer vision — it's not arbitrary, it's statistically principled.

#### Reduces to 1D Optimization
Although \(\hat{\mathbf{X}}\) has 3 unknowns \([X, Y, Z]\), the constraint that \(\hat{\mathbf{x}}\) and \(\hat{\mathbf{x}}'\) must lie on corresponding epipolar lines reduces the search to a single scalar parameter \(t\) — yielding a closed-form polynomial root-finding solution (Hartley & Sturm, 1997).

#### Algebraic vs. Geometric Error

| | Algebraic Error (SVD) | Geometric Error (Method 3) |
|--|---|---|
| **Minimizes** | \(\|\mathbf{AX}\|^2\) | \(d(\mathbf{x}, \hat{\mathbf{x}})^2 + d(\mathbf{x}', \hat{\mathbf{x}}')^2\) |
| **Units** | Dimensionless, camera-parameter-dependent | **Pixels** |
| **Physical meaning** | None | Direct reprojection error |
| **Statistical basis** | None | **MLE under Gaussian noise** |
| **Speed** | Fast, closed-form | Requires iteration |

#### Warm Start Strategy
In practice, both methods are combined:
1. **SVD first** — fast closed-form algebraic solution gives a good initial estimate \(\mathbf{X}_\text{SVD}\)
2. **Geometric refinement** — use \(\mathbf{X}_\text{SVD}\) as starting point for iterative optimization (e.g., **Levenberg-Marquardt**)

Starting close to the answer means: fewer iterations, avoids bad local minima, converges in typically 2–5 steps.

***

## 9. Full Pipeline Summary

```
Two images I₁, I₂ from calibrated cameras P, P'
            ↓
   [Rectification if cameras are converging]
   Apply homography H₁, H₂ → parallel geometry
            ↓
   [Correspondence]
   For each pixel i in I₁:
     → cross-correlate W₁(i) along epipolar line in I₂
     → pick disparity D(i) = argmax correlation
     → OR: globally solve energy minimization via graph cuts
            ↓
   [Disparity → Depth]
   Z = bf / D(i)   for each pixel
            ↓
   [Triangulation for sparse/exact 3D points]
   1. Form system AX = 0 using cross-product trick
   2. Solve via SVD → X_SVD
   3. Refine via geometric MLE (LM optimizer) → X̂_final
            ↓
   Dense depth map D  +  Sparse 3D point cloud {X̂}
```