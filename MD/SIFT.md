# SIFT & Keypoint Detection

---

## 1. The Core Problem

When a computer tries to recognize the same object in two photos — taken at different distances, rotations, or lighting — the pixel patterns look completely different. **SIFT (Scale-Invariant Feature Transform)** solves this by finding **keypoints**: distinctive, stable locations that can be reliably detected and matched regardless of scale, rotation, or illumination.

***

## 2. Scale Space Representation

### The Formula

$$ L(x, y, \sigma) = G(x, y, \sigma) * I(x, y) $$

- $I(x,y)$ → original image
- $G(x,y,\sigma)$ → 2D Gaussian kernel with standard deviation $\sigma$
- $*$ → convolution
- $L(x,y,\sigma)$ → blurred image at scale $\sigma$
- $\sigma$ controls **what size features you are sensitive to**

### What $\sigma$ Controls

| $\sigma$ | Blur | Details Visible |
|---|---|---|
| Small | Light | Fine details — edges, textures |
| Large | Heavy | Coarse details — large blobs, shapes |

Large $\sigma$ corresponds to looking at the picture from further away. Small structures vanish; only dominant shapes survive.

### Why a Stack of Images?

A single blur level is not enough: different features come into focus at different $\sigma$. We therefore build a **family of blurred images** at increasing $\sigma$ — a continuous **scale space** $L(x, y, \sigma)$. Each feature has a *characteristic scale* at which it is most prominent and fades away elsewhere; a keypoint will be located as an **extremum across $\sigma$** of a scale-normalised response built from this stack (formalised via the Laplacian / Difference-of-Gaussians in §4).

The remaining question is *how to build the stack efficiently*. The naive recipe — convolve the original image with a sequence of progressively wider Gaussians — turns out to be prohibitively expensive, as the next section quantifies.

***

## 3. Scale Space Pyramid

### Why Not Just Increase $\sigma$ Directly?

A Gaussian kernel of width $\sigma$ requires a spatial support of size $\approx 6\sigma \times 6\sigma$, so convolution at scale $\sigma$ costs $O(\sigma^2 \cdot W \cdot H)$ operations:

| $\sigma$ | Kernel Size | Operations/pixel |
|---|---|---|
| 2 | 12×12 | 144 |
| 10 | 60×60 | 3,600 |
| 20 | 180×180 | 32,400 |

Using scales alone is **inefficient and numerically unstable**. The pyramid solves this.

### Key Equivalence

> **Blurring a full-resolution image with $\sigma=16$ is mathematically equivalent to blurring a 2× downsampled image with $\sigma=8$.**

When you halve image dimensions, all spatial distances halve, so the same physical scale is represented by half the $\sigma$. Small, cheap kernels on small images do the exact same job as huge kernels on the original — not a compromise, an exact equivalence.

**Aliasing caveat.** Naive 2× subsampling would fold frequencies above the new Nyquist limit back as low-frequency ghosts, creating spurious keypoints. The fix is to **blur before subsampling**: the Gaussian at the seed level already has $\sigma = 2\sigma_0$, wide enough to act as the anti-alias filter that kills everything above the post-downsample Nyquist limit. Subsampling then drops no recoverable information, so the equivalence above remains exact (detailed in *Downsampling: How It Works* below).

### Octaves

An **octave** is a level where the image resolution is halved:

| Octave | Image Size |
|---|---|
| 0 | 512×512 |
| 1 | 256×256 |
| 2 | 128×128 |
| 3 | 64×64 |

### Scales Within Each Octave

$$ \sigma_i = \sigma_0 \cdot k^i, \quad k = 2^{1/s} $$

- $\sigma_0 = 1.6$ → base scale
- $s = 3$ → scales per octave (controls precision of scale detection)
- $k = 2^{1/3} \approx 1.26$ → scale multiplier per level

**For $s=3$, the 6 blurred images in Octave 0:**

| Level | $\sigma$ | Role |
|---|---|---|
| 0 | 1.60 | First scale |
| 1 | 2.02 | |
| 2 | 2.54 | |
| 3 | 3.20 | **Seed for next octave** (= $2\sigma_0$) |
| 4 | 4.03 | DoG buffer only |
| 5 | 5.08 | DoG buffer only |

### Seeding the Next Octave

Level $s$ has $\sigma = 2\sigma_0$. When downsampled by 2×, the effective $\sigma$ becomes $\sigma_0$ in the new coordinate system — so the next octave starts at the same base scale at half resolution. The pyramid is **seamlessly continuous** across octaves.

**Local vs. absolute $\sigma$.** Inside every octave the *local* $\sigma$ sequence is identical: $\sigma_0 \to 2\sigma_0$ via $\sigma_i = \sigma_0 \cdot k^i$. What changes is the pixel grid — each octave's pixels cover twice the physical distance of the previous one — so in **absolute (original-image) units** the $\sigma$ range *doubles every octave*:

$$\sigma_{\text{abs}}(o, i) \;=\; 2^{o} \cdot \sigma_0 \cdot k^i, \qquad k = 2^{1/s}.$$

The geometric doubling of blur per octave is thus achieved *for free* by subsampling, not by growing kernels — and every octave runs at the same low computational cost.

### Optimal Number of Octaves

$$ \text{Octaves} = \log_2(\min(W, H)) - C $$

- **W, H** → image width and height in pixels
- **C** → small constant (typically 3), stops analysis when the image becomes too small (below ~8×8) to yield meaningful features
- Example: 512×512 image → $\log_2(512) - 3 = 6$ octaves
- More octaves → ability to detect larger structures

### Downsampling: How It Works

1. **Blur first** — existing Gaussian blur at level $s$ removes all frequencies above the new Nyquist limit → prevents **aliasing**
2. **Subsample** — keep every 2nd pixel: $I_{\text{down}}[i,j] = I[2i, 2j]$

**Why blur before downsampling?** Without blurring, high-frequency patterns get **folded back** as false low-frequency artifacts. E.g., a pattern repeating every 1.5 pixels appears to repeat every 3 pixels after naive downsampling — ghost patterns that generate false keypoints. Gaussian blur destroys all frequencies above the new Nyquist limit before you drop pixels, so there is nothing left to fold back.

***

## 4. Difference of Gaussians (DoG)

### Why DoG?

The Laplacian of Gaussian (LoG) is the ideal blob detector but expensive. DoG approximates it cheaply. From the heat-equation identity $\partial G_\sigma / \partial \sigma = \sigma\, \nabla^2 G_\sigma$, a finite difference between adjacent scales gives

$$D_i \;=\; L(\sigma_{i+1}) - L(\sigma_i) \;=\; L(k\sigma_i) - L(\sigma_i) \;\approx\; (k - 1)\,\underbrace{\sigma_i^2\, \nabla^2 G_{\sigma_i} \ast I}_{\text{scale-normalised LoG at } \sigma_i}.$$

So **the DoG between scales $\sigma_i$ and $\sigma_{i+1} = k\sigma_i$ approximates the scale-normalised LoG at the smaller scale $\sigma_i$**, up to the constant factor $(k - 1)$. The constant is the same for every $i$, so it does not affect the location of extrema across $(x, y, \sigma)$ — DoG extrema and LoG extrema coincide.

DoG measures **how much a pixel differs from its surroundings** — large response at blob centers, weak response in flat regions.

### How Many Images Per Octave?

Working backwards:
- Need **$s$ usable DoG images** for extrema detection
- Each DoG needs a neighbor above and below → need **$s+2$ DoG images** (1 buffer top, 1 bottom)
- $n$ blurred images produce $n-1$ DoG images → need **$s+3$ blurred images**

For $s=3$: **6 blurred → 5 DoG → search middle 3 for extrema**

***

## 5. Keypoint Detection — Extrema in DoG

### The Algorithm

```
for each octave:
    for i = 1 to (num_DoG - 2):   # skip first and last
        at position (x, y, i):
            compare with 26 neighbors:
              - 8 spatial (same DoG layer)
              - 9 in layer i+1 (above)
              - 9 in layer i-1 (below)
        if local maximum or minimum across all 26:
            → candidate keypoint
```

### Why an Extremum = Keypoint

- **Spatial maximum** → dark blob on bright background
- **Spatial minimum** → bright blob on dark background
- **Scale extremum** → feature is best explained at this specific $\sigma$

A small blob produces a strong DoG response at small $\sigma$ (Gaussian matches its size); at large $\sigma$ it blurs away → no extremum. Each feature resonates at its own natural scale. 

Being an extremum in all 26 neighbors means: *spatially distinctive + scale-matched* — exactly what a good keypoint needs.

***

## 6. Keypoint Refinement via Taylor Expansion

### Problem

Keypoints are detected in discrete space, but true extrema exist in continuous space.

### Taylor Expansion (3D Multivariate)

The DoG function $D(x,y,\sigma)$ is expanded around the detected discrete keypoint $(x_i, y_i, \sigma_i)$:

$$ D(\Delta) = D_0 + \nabla D^T \Delta + \frac{1}{2} \Delta^T H \Delta $$

Where the **offset vector** $\Delta \in \mathbb{R}^{3\times1}$:

$$ \Delta = \begin{pmatrix} x - x_i \\ y - y_i \\ \sigma - \sigma_i \end{pmatrix} $$

**Gradient vector** $\nabla D \in \mathbb{R}^{3\times1}$ — direction of steepest increase, evaluated at the keypoint:

$$ \nabla D = \begin{pmatrix} \frac{\partial D}{\partial x} \\ \frac{\partial D}{\partial y} \\ \frac{\partial D}{\partial \sigma} \end{pmatrix} $$

**Hessian matrix** $H \in \mathbb{R}^{3\times3}$ — curvature of DoG in all three directions, symmetric ($D_{xy}=D_{yx}$, etc.):

$$ H = \begin{pmatrix} D_{xx} & D_{xy} & D_{x\sigma} \\ D_{yx} & D_{yy} & D_{y\sigma} \\ D_{\sigma x} & D_{\sigma y} & D_{\sigma\sigma} \end{pmatrix} $$

### Hessian Values via Finite Differences

All computed directly from existing DoG images — no extra computation:

$$ D_{xx} = D(x+1,y) - 2D(x,y) + D(x-1,y) $$
$$ D_{yy} = D(x,y+1) - 2D(x,y) + D(x,y-1) $$
$$ D_{xy} = \frac{D(x+1,y+1) - D(x-1,y+1) - D(x+1,y-1) + D(x-1,y-1)}{4} $$
$$ D_{\sigma\sigma} = D_{i+1}(x,y) - 2D_i(x,y) + D_{i-1}(x,y) $$
$$ D_{x\sigma} = \frac{D_{i+1}(x+1,y) - D_{i+1}(x-1,y) - D_{i-1}(x+1,y) + D_{i-1}(x-1,y)}{4} $$

### Solving for the Refined Location

Differentiate w.r.t. $\Delta$ and set to zero:

$$ \nabla D + H\Delta = 0 $$

$$ \boxed{\Delta = -H^{-1} \nabla D = -\begin{pmatrix} D_{xx} & D_{xy} & D_{x\sigma} \\ D_{yx} & D_{yy} & D_{y\sigma} \\ D_{\sigma x} & D_{\sigma y} & D_{\sigma\sigma} \end{pmatrix}^{-1} \begin{pmatrix} D_x \\ D_y \\ D_\sigma \end{pmatrix}} $$

This 3D offset gives sub-pixel correction in **x, y, and scale simultaneously**.

### Updating the Keypoint

$$ \begin{pmatrix} x \\ y \\ \sigma \end{pmatrix}_{\text{refined}} = \begin{pmatrix} x_i \\ y_i \\ \sigma_i \end{pmatrix} + \begin{pmatrix} \Delta_x \\ \Delta_y \\ \Delta_\sigma \end{pmatrix} $$

**Rule:** If $|\Delta| > 0.5$ in any dimension → true extremum lies in a neighboring pixel/scale → move there and repeat. Otherwise accept.

### Value at Refined Location

$$ D(\hat{\Delta}) = D_0 + \frac{1}{2}\nabla D^T \Delta = D_0 + \frac{1}{2}\begin{pmatrix} D_x & D_y & D_\sigma \end{pmatrix} \begin{pmatrix} \Delta_x \\ \Delta_y \\ \Delta_\sigma \end{pmatrix} $$

***

## 7. Rejecting Bad Keypoints

### 1. Low Contrast Rejection

$$ |D(\hat{\Delta})| < 0.03 \Rightarrow \textbf{Reject} $$

Small DoG response → weak feature → unstable and noise-sensitive.

### 2. Edge Response Rejection

Edges have high gradient in one direction, low in the other → not distinctive, not stable. Use the **2×2 spatial Hessian** of the DoG surface at the keypoint:

$$ H = \begin{bmatrix} D_{xx} & D_{xy} \\ D_{yx} & D_{yy} \end{bmatrix} $$

**Geometric meaning of the eigenvalues.** $\lambda_1, \lambda_2$ of $H$ are the **maximal and minimal principal curvatures** of the DoG surface $D(x, y)$ at that point. Hence:

- **Edge** — high *maximal* curvature (across the edge), very low *minimal* curvature (along it) ⇒ $\lambda_1 \gg \lambda_2$.
- **Corner / blob keypoint** — high curvature in *both* principal directions ⇒ $\lambda_1 \approx \lambda_2$, both large.

The ratio test below is therefore a *principal-curvature ratio* test.

Ratio test (avoids computing eigenvalues directly) — the **acceptance** condition:

$$ \frac{(\text{Tr}(H))^2}{\text{Det}(H)} < \frac{(r+1)^2}{r} \;\Rightarrow\; \textbf{Keep}, \qquad \text{else } \textbf{Reject} \quad (r = 10 \text{ typically}). $$

Where $\text{Tr}(H) = \lambda_1 + \lambda_2$, $\text{Det}(H) = \lambda_1\lambda_2$:

- **Ratio below threshold** (small) → $\lambda_1 \approx \lambda_2$, eigenvalues balanced → corner / blob → **keep**.
- **Ratio above threshold** (large) → one $\lambda$ dominates → edge-like → **reject**.

***

## 8. Orientation Assignment

### Problem

Same keypoint in a rotated image produces different gradient directions → descriptors won't match without correction.

### Step 1 — Compute Gradients in Neighborhood

Using **$L(x,y,\sigma_{\text{keypoint}})$** — the blurred image at the scale where the keypoint was detected (the scale of the DoG layer where the extremum was found, not the DoG itself — DoG is a difference image, less suitable for gradient computation):

$$ m(x,y) = \sqrt{(L(x+1,y)-L(x-1,y))^2 + (L(x,y+1)-L(x,y-1))^2} $$
$$ \theta(x,y) = \tan^{-1}\!\left(\frac{L(x,y+1)-L(x,y-1)}{L(x+1,y)-L(x-1,y)}\right) $$

Computed for all pixels within a **circular window of radius $\approx 1.5\sigma$**. Number of pixels scales as $O(\sigma^2)$.

### Step 2 — Build Weighted Orientation Histogram

- Weight each pixel: $w(x,y) = G(x,y,\ 1.5\sigma)$ — pixels closer to keypoint contribute more
- Each pixel votes $w(x,y) \times m(x,y)$ into the **bin of $\theta(x,y)$**
- **36 bins × 10°** = full 360°

### Step 3 — Find Dominant Orientation

$$ \theta^* = \text{argmax(histogram)} $$

**Multiple orientations:** If any other peak $\geq 0.8 \times \text{max peak}$ → create a **separate keypoint** for each orientation at same location/scale. (~15% of keypoints get this treatment — significantly improves matching reliability.)

***

## 9. Descriptor Construction

### What is a Descriptor?

$$ \text{Descriptor} \in \mathbb{R}^{128} $$

Encodes local image structure — gradient distribution, spatial layout, orientation patterns. If two descriptors match → same physical point.

### Step 1 — Take 16×16 Window

Centered on keypoint, **physically rotated by $-\theta^*$** so the grid aligns with the dominant orientation.

### Step 2 — Split into 4×4 Grid of Cells

16 cells, each 4×4 pixels.

### Step 3 — Per-Pixel Computation (All 256 Pixels)

For each pixel:

1. $m(x,y)$ ← gradient magnitude (finite differences on $L$)
2. $\theta(x,y)$ ← gradient orientation
3. $w(x,y)$ ← Gaussian weight **w.r.t. keypoint center** (not cell center):
$$ w(x,y) = \exp\!\left(-\frac{(x-x_k)^2+(y-y_k)^2}{2(1.5\sigma)^2}\right) $$
4. $\theta_{\text{rel}} = \theta(x,y) - \theta^*$ ← rotated angle
5. Vote $w \times m$ into bin of $\theta_{\text{rel}}$ of the pixel's cell

**Binning logic:**
- **Which cell** → determined by pixel's spatial position in the 16×16 grid
- **Which bin within the cell** → determined purely by $\theta_{\text{rel}}$, not spatial position within the cell

### Step 4 — Per-Cell 8-Bin Histogram

Each bin covers 45°. Each entry = total accumulated $w \times m$ from all pixels in that cell whose gradient pointed in that direction. Pixels near the keypoint center get the highest $w$, so central cells dominate over corner cells.

### Step 5 — Concatenate → 128D

$$ 16 \text{ cells} \times 8 \text{ bins} = 128\text{D vector} $$

**General rule:** Descriptor size $= p \times n^2$ where $p$ = bins per cell, $n^2$ = number of cells.

### Step 6 — Normalize

$$ \hat{\mathbf{d}} = \frac{\mathbf{d}}{\|\mathbf{d}\|} \Rightarrow \|\hat{\mathbf{d}}\| = 1 $$

**Illumination invariance — what this buys and what it doesn't:**

- **Multiplicative ($I \to a I$) — invariant.** All gradient magnitudes $m$ scale by $a$, so $\mathbf{d} \to a\mathbf{d}$, but after $L_2$ normalisation $\hat{\mathbf{d}}$ is unchanged.
- **Additive ($I \to I + b$) — invariant for free.** The descriptor is built from *gradients*, which kill any constant offset before the histogram is ever formed.
- **Non-linear illumination (gamma, saturation, shadows) — NOT invariant.** $L_2$ normalisation only undoes a *global scalar* change in intensity. Lowe's fix is to clip each $\hat{d}_i$ at $0.2$ (suppress the few dominant gradients that saturate after non-linear changes) and re-normalise.

### Why Cells?

A single global histogram tells you **what directions** exist but not **where**. Cells preserve spatial locality — gradient on the left edge is recorded separately from the right. This gives the descriptor its discriminative power.

***

## 10. Rotation Invariance — Full Picture

Two simultaneous rotations by $-\theta^*$:

1. **Spatial rotation** — the 16×16 sampling grid is physically rotated to align with $\theta^*$
2. **Angle rotation** — every gradient angle: $\theta \rightarrow \theta - \theta^*$

Magnitudes $m$ are not rotated — directionless scalars. Gradient computation is unaffected — $m$ and $\theta$ are measured first from raw pixel differences; rotation only changes how they are **binned**.

**Result:** Both cell layout and binned angles are relative to the dominant direction → 128D descriptor is identical regardless of image rotation.

***

## 11. Keypoint Matching Across Images

### Step 1 — Nearest Neighbor in Descriptor Space

For each descriptor $\mathbf{x}_i \in \mathbb{R}^{128}$ in Image 1, compute Euclidean distance to every descriptor $\mathbf{y}_j \in \mathbb{R}^{128}$ in Image 2:

$$ d(\mathbf{x}_i, \mathbf{y}_j) = \|\mathbf{x}_i - \mathbf{y}_j\|_2 $$

### Step 2 — Lowe's Ratio Test

Simply taking the nearest neighbor is unreliable — a feature may have no correct match (occluded) yet still return a nearest neighbor. Lowe's fix: find the **two closest** descriptors and compute:

$$ \text{ratio} = \frac{d_{\text{nearest}}}{d_{\text{second nearest}}} $$

- **Ratio < 0.8** → best match clearly stands out → **accept**
- **Ratio ≥ 0.8** → two candidates nearly equal → ambiguous (e.g., repetitive texture like bricks) → **reject**

**Why not an absolute threshold?** The ratio is self-normalizing — it doesn't break when lighting or angle changes shift all distances uniformly.

**Concrete example:**
- Unique corner: $d_1=0.3$, $d_2=0.9$ → ratio $0.33$ → accept ✓
- Repetitive bricks: $d_1=0.75$, $d_2=0.78$ → ratio $0.96$ → reject ✗

Eliminates ~90% of false matches while retaining ~95% of correct ones.

### Step 3 — Geometric Verification (RANSAC)

Surviving matches are verified by fitting a homography or fundamental matrix. Matches geometrically inconsistent with the estimated transformation are discarded as outliers.

***

## 12. End-to-End SIFT Pipeline

```
Input Image
    ↓
Build scale space pyramid
  → s+3 blurred images per octave (for s=3: 6 images)
  → σᵢ = σ₀ · kⁱ,  k = 2^(1/s),  σ₀ = 1.6
  → Downsample by 2× at end of each octave (blur first → subsample)
    ↓
Compute DoG (s+2 per octave → 5 DoG images for s=3)
    ↓
Detect extrema across 26-neighbor (x, y, σ) volume
→ candidate keypoints
    ↓
Refine via 3D Taylor expansion:
  Δ = -H⁻¹ ∇D  →  sub-pixel correction in (x, y, σ)
    ↓
Reject:
  |D(Δ̂)| < 0.03           (low contrast)
  (Tr(H))²/Det(H) ≥ (r+1)²/r  (edge response)
    ↓
Assign orientation(s) via 36-bin histogram → θ*
  (create multiple keypoints if peak ≥ 0.8 × max)
    ↓
Build 128D descriptor:
  - Rotate 16×16 grid by -θ*
  - 4×4 cells × 8-bin histogram
  - Vote: w(x,y) × m(x,y) into bin of (θ - θ*)
  - Normalize to unit length
    ↓
Match via nearest neighbor + Lowe's ratio test (< 0.8)
    ↓
Geometric verification via RANSAC
    ↓
Applications: panorama stitching · 3D reconstruction
              object recognition · robot localization
              video tracking · medical image registration
```

