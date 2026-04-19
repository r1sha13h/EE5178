# Blob Detection — Comprehensive Notes
*EE5178: Modern Computer Vision | Based on Forsyth & Ponce*

***

## 1. What is a Blob?

A **blob** is a compact, roughly circular image region with uniform intensity distinct from its background. Unlike **corners** (intensity changes in all directions), blobs are characterized by their *spatial extent and scale*, making them ideal keypoints for scale-invariant tasks.

***

## 2. Applications of Keypoint Detection

Keypoints (corners + blobs) underpin:
- Panorama stitching and image alignment
- 3D reconstruction and motion tracking
- Object recognition and database retrieval
- Robot navigation

### Properties of Good Keypoints
| Property | Meaning |
|---|---|
| **Repeatability** | Detected consistently across viewpoints and lighting |
| **Saliency** | Each feature has a distinctive, unique descriptor |
| **Compactness** | Far fewer features than total pixels |
| **Locality** | Occupies a small image region; robust to occlusion |

***

## 3. The Blob Filter: Laplacian of Gaussian (LoG)

### 3.1 Definition

The LoG is a circularly symmetric 2nd-order differential operator:

\[
\nabla^2 g = \frac{\partial^2 g}{\partial x^2} + \frac{\partial^2 g}{\partial y^2}
\]

Explicitly in 2D:

\[
\text{LoG}(x, y, \sigma) = \frac{1}{\pi\sigma^4}\left(1 - \frac{x^2 + y^2}{2\sigma^2}\right)e^{-(x^2+y^2)/2\sigma^2}
\]

In polar form (\( \rho^2 = x^2 + y^2 \)):

\[
\text{LoG}(\rho, \sigma) \propto (\rho^2 - 2\sigma^2)\, e^{-\rho^2/(2\sigma^2)}
\]

Its shape is the **"Mexican hat"** — positive central lobe, negative surround.

### 3.2 From Edges to Blobs

- In **1D**, a single Laplacian response detects an **edge** (zero-crossing of 2nd derivative)
- A **blob** = superposition of two back-to-back edges (two ripples)
- The Laplacian achieves a **spatial maximum at the blob center** when σ is matched to the blob's scale

***

## 4. The Scale Normalization Problem

### 4.1 Why Response Decays with σ

For a step edge \( f(x) = \mathbb{1}[x \geq 0] \), convolving with the 1st derivative of Gaussian gives:

\[
R(x, \sigma) = \frac{1}{\sigma\sqrt{2\pi}} e^{-x^2/2\sigma^2}
\]

The **peak response** at the edge \( (x=0) \) is:

\[
R(0, \sigma) = \frac{1}{\sigma\sqrt{2\pi}}
\]

This decays as \( 1/\sigma \). Physically: a wider Gaussian spreads energy over a larger area. Since total area is conserved (\( \int R\, dx = 1\, \forall\, \sigma \)), a wider filter must have a shorter peak. The same argument gives \( 1/\sigma^3 \) decay for the 2nd derivative (Laplacian).

### 4.2 The Fix: Scale Normalization

Multiply derivatives by a power of σ to compensate:

| Filter | Normalization factor |
|---|---|
| 1st derivative (gradient) | × **σ** |
| 2nd derivative (Laplacian) | × **σ²** |

**Scale-normalized Laplacian:**

\[
\nabla^2_{\text{norm}} g = \sigma^2 \left(\frac{\partial^2 g}{\partial x^2} + \frac{\partial^2 g}{\partial y^2}\right)
\]

Now the peak response is **independent of σ**, enabling fair comparison across scales.

***

## 5. Optimal Scale for a Circular Blob

The LoG zero-crossing occurs at \( \rho = \sigma\sqrt{2} \). For maximum response to a binary circle of radius \( r \), align the LoG zero-crossing with the circle boundary — positive lobe inside, negative surround outside:

\[
\sigma\sqrt{2} = r \implies \boxed{\sigma^* = \frac{r}{\sqrt{2}}}
\]

Conversely, a detected extremum at scale \( \sigma^* \) corresponds to a blob of radius:

\[
r = \sigma^* \cdot \sqrt{2}
\]

This is why detected blobs are visualized as **circles of varying sizes** — larger circles indicate features detected at coarser scales.

***

## 6. Scale-Space Blob Detection Algorithm

### Step 1 — Build Scale-Space
Compute the **scale-normalized LoG** response at multiple scales \( \sigma_1, \sigma_2, \ldots \), stacking them into a 3D scale-space volume \((x, y, \sigma)\).

### Step 2 — Detect 3D Extrema
Find local maxima/minima of squared LoG response. Each candidate point is compared to its **26 neighbors**:
- 8 spatial neighbors (same scale layer)
- 9 neighbors in the scale layer above
- 9 neighbors in the scale layer below

If the point is strictly the extremum among all 26, it is a keypoint with location \((x, y)\) and characteristic scale \(\sigma^*\).

***

## 7. Efficient Implementation: Difference of Gaussians (DoG)

The LoG is computationally expensive. **DoG** is its efficient approximation:

\[
\text{DoG}(x, y, \sigma) = G(x, y, k\sigma) - G(x, y, \sigma) \approx (k-1)\sigma^2\,\nabla^2 g
\]

So DoG ≈ scale-normalized LoG, up to the constant factor \((k-1)\).

### Which Scale Does a Detected Blob Belong To?

\( G(k\sigma) - G(\sigma) \) approximates the LoG at scale **σ** (the smaller of the two). So the detected blob is assigned scale \( \sigma \), and its radius is \( \sigma\sqrt{2} \).

### DoG Pyramid Construction *(Lowe, IJCV 2004)*
1. Build a **Gaussian pyramid**: blur the image at multiple σ levels within each octave
2. **Subtract adjacent Gaussian layers** → DoG images
3. Search the DoG stack for 3D extrema

The absolute scale of a keypoint across octaves is:

\[
\sigma_{\text{absolute}} = \sigma_0 \cdot k^i \cdot 2^{\text{octave}}, \quad k = 2^{1/s}
\]

where \( s \) = number of scale levels per octave, \( i \) = layer index.

***

## 8. SIFT Descriptor

Once blobs are localized, each is described using **SIFT**:

1. Take a **16×16 patch** around the keypoint, divided into a **4×4 grid** of cells
2. Compute an **8-bin gradient orientation histogram** per cell
3. Concatenate → **128-dimensional descriptor** (4×4×8)

### Invariance Properties
- **Rotation:** Compute the dominant gradient orientation around the keypoint, then **rotate the patch to align with it** before computing the descriptor — same blob rotated in two images yields identical descriptors
- **Illumination:** **L2-normalize** the 128-dim vector to unit length — cancels multiplicative brightness changes; gradients are inherently invariant to additive intensity offsets

***

## 9. Full Pipeline Summary

```
Image(s)
   ↓
1. Blob Detection  →  Scale-normalized DoG pyramid at multiple σ
                  →  Find 3D extrema in (x, y, σ) scale-space
   ↓
2. Keypoint Description  →  128-dim SIFT descriptor per blob
                          →  Rotation + illumination normalized
   ↓
3. Feature Matching  →  Nearest-neighbor matching in descriptor space
   ↓
4. Geometric Alignment  →  Homography / RANSAC for stitching / reconstruction
```