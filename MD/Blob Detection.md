# Blob Detection

*EE5178 — Modern Computer Vision*

---

## 1. What Is a Blob?

A **blob** is a compact, roughly circular image region of approximately uniform intensity that is distinct from its background. Whereas **corners** are characterised by intensity changes in *all* directions, blobs are defined by their **spatial extent and scale**, making them ideal keypoints for scale-invariant tasks such as matching across zoom levels.

---

## 2. Keypoint Detection — Motivation

### 2.1 Applications

Keypoints — corners and blobs — underpin a wide range of computer-vision tasks:

- Panorama stitching and image alignment.
- 3D reconstruction and motion tracking.
- Object recognition and database retrieval.
- Robot navigation.

### 2.2 Properties of a Good Keypoint

| Property | Meaning |
|---|---|
| **Repeatability** | Detected consistently across viewpoint, scale, and lighting changes. |
| **Saliency** | Each feature has a distinctive, locally unique descriptor. |
| **Compactness** | Far fewer features than pixels — efficient downstream processing. |
| **Locality** | Occupies a small image region — robust to occlusion and clutter. |

---

## 3. The Blob Filter — Laplacian of Gaussian (LoG)

> **Basic idea.** To detect blobs, convolve the image with a *blob filter* at multiple scales and look for **extrema** (both maxima *and* minima) of the filter response in the resulting **scale-space** $(x, y, \sigma)$. A blob is therefore localised by **two simultaneous selections**:
>
> - **Spatial selection** — *where* in the image the response peaks (the blob centre).
> - **Scale selection** — *at which $\sigma$* the response peaks (the blob's characteristic size).
>
> The remainder of this section motivates the choice of filter; §4 makes scale selection well-posed via normalisation; §6 turns this into a concrete algorithm.

### 3.1 Definition

The Laplacian is a circularly symmetric second-order differential operator:

$$\nabla^2 g = \frac{\partial^2 g}{\partial x^2} + \frac{\partial^2 g}{\partial y^2}.$$

Applied to a 2D Gaussian $g_\sigma$ it gives the **Laplacian-of-Gaussian** kernel, with closed form

$$\mathrm{LoG}(x, y, \sigma) = \frac{1}{\pi \sigma^4}\left(1 - \frac{x^2 + y^2}{2 \sigma^2}\right) e^{-(x^2 + y^2) / 2 \sigma^2}.$$

In polar coordinates with $\rho^2 = x^2 + y^2$,

$$\mathrm{LoG}(\rho, \sigma) \;\propto\; (\rho^2 - 2 \sigma^2)\, e^{-\rho^2 / 2 \sigma^2}.$$

Its profile is the **"Mexican hat" (sombrero)** — a positive central lobe surrounded by a negative ring (under the sign convention above).

### 3.2 From Edges to Blobs — Spatial Selection

- In **1D**, a zero-crossing of the Laplacian marks an **edge** — i.e. an edge produces a single *ripple* in the Laplacian profile.
- A **blob** is heuristically a *superposition of two back-to-back edges*, so its Laplacian profile is a **pair of ripples** that reinforce one another at the centre.
- **Spatial selection.** The magnitude of the Laplacian response attains a **maximum at the blob centre**, *provided the scale of the Laplacian is matched to the scale of the blob*. Mismatch in $\sigma$ shifts the ripples apart and the central reinforcement collapses — motivating the scale-selection problem of §4.

---

## 4. The Scale-Normalisation Problem

**Scale selection** asks: among Laplacian responses at $\sigma_1 < \sigma_2 < \cdots$, which scale gives the maximum at a given pixel? Naively, just pick $\arg\max_\sigma \lvert L_\sigma * I \rvert$. The catch: even for a *fixed* signal, the raw Laplacian response **decays monotonically with $\sigma$**, so the argmax is biased toward the smallest scale and tells us nothing about the blob's true size. Section 4.1 quantifies the decay; §4.2 fixes it.

### 4.1 Why the Raw Response Decays with $\sigma$

**1D Gaussian and its derivatives** (reference forms used below):

$$g(x, \sigma) = \frac{1}{\sigma\sqrt{2\pi}}\, e^{-x^2 / 2\sigma^2}, \qquad \frac{\partial g}{\partial x} = -\frac{x}{\sigma^3\sqrt{2\pi}}\, e^{-x^2 / 2\sigma^2}, \qquad \frac{\partial^2 g}{\partial x^2} = -\frac{\sigma^2 - x^2}{\sigma^5\sqrt{2\pi}}\, e^{-x^2 / 2\sigma^2}.$$

Consider an idealised 1D step edge $f(x) = \mathbb{1}[x \geq 0]$ filtered with derivatives of a Gaussian. By the derivative theorem of convolution, derivatives can be transferred onto the kernel:

$$\frac{d^k}{dx^k}(f * G_\sigma)(x) \;=\; (f * G_\sigma^{(k)})(x) \;=\; (f^{(k)} * G_\sigma)(x).$$

**First derivative (gradient).** For a step, $f' = \delta$, so

$$R_1(x, \sigma) \;=\; (f * G_\sigma')(x) \;=\; (\delta * G_\sigma)(x) \;=\; G_\sigma(x) \;=\; \frac{1}{\sigma\sqrt{2\pi}}\, e^{-x^2 / 2\sigma^2}.$$

The peak sits at the edge ($x = 0$):

$$R_1(0, \sigma) \;=\; \frac{1}{\sigma\sqrt{2\pi}} \;\propto\; \frac{1}{\sigma}.$$

**Second derivative (Laplacian).** Pushing one derivative onto the signal and one onto the kernel,

$$R_2(x, \sigma) \;=\; (f * G_\sigma'')(x) \;=\; (\delta * G_\sigma')(x) \;=\; G_\sigma'(x) \;=\; -\frac{x}{\sigma^3\sqrt{2\pi}}\, e^{-x^2 / 2\sigma^2}.$$

Setting $dR_2/dx = 0$ gives extrema at $x = \pm\sigma$, so the peak magnitude is

$$\lvert R_2(\pm\sigma, \sigma) \rvert \;=\; \frac{1}{\sigma^2\sqrt{2\pi}}\, e^{-1/2} \;\propto\; \frac{1}{\sigma^2}.$$

**Physical reason.** The Gaussian is area-preserving ($\int G_\sigma\, dx = 1$ for every $\sigma$), so a wider kernel must spread its unit mass over a larger support, lowering its peak by $1/\sigma$. Each additional spatial derivative differentiates the exponent $-x^2/2\sigma^2$, pulling out one more factor of $1/\sigma$. The $k$-th derivative of $G_\sigma$ therefore scales as $\sigma^{-(k+1)}$ at its extrema, so the response to a step decays as $1/\sigma^{k}$ — requiring a compensating $\sigma^{k}$ in the normalisation.

### 4.2 The Fix — Scale Normalisation

To make responses comparable across scales, multiply each derivative by a compensating power of $\sigma$ (Lindeberg, 1998). The factors below are derived from Lindeberg's $\gamma$-normalisation framework, which optimises for Gaussian-matched signals (blobs) rather than step edges — the step-edge analysis in §4.1 is heuristic motivation, not a rigorous derivation:

| Filter | Normalisation factor | Reason |
|---|---|---|
| 1st derivative (gradient) | $\times \sigma$ | Cancels $1/\sigma$ peak decay |
| 2nd derivative (Laplacian) | $\times \sigma^2$ | Laplacian = 2nd Gaussian derivative → cancels $1/\sigma^2$ decay |

The **scale-normalised Laplacian** is

$$\nabla^2_{\mathrm{norm}}\, g \;=\; \sigma^2 \left(\frac{\partial^2 g}{\partial x^2} + \frac{\partial^2 g}{\partial y^2}\right).$$

Under this normalisation the peak response to a matched blob becomes **independent of $\sigma$**, enabling fair comparison across scales — the cornerstone of scale-space analysis.

---

## 5. Optimal Scale for a Circular Blob

The LoG has its zero-crossing at $\rho = \sigma\sqrt{2}$. For a binary disc of radius $r$ the response is **maximised** when the LoG's zero-crossing aligns with the disc boundary — placing the positive lobe inside the disc and the negative ring outside:

$$\sigma\sqrt{2} = r \;\Longrightarrow\; \boxed{\;\sigma^{*} = \frac{r}{\sqrt{2}}.\;}$$

Conversely, a scale-space extremum detected at scale $\sigma^{*}$ corresponds to a blob of radius

$$r = \sigma^{*}\, \sqrt{2}.$$

This is why detected blobs are conventionally visualised as **circles of varying sizes** — larger circles indicate features detected at coarser scales.

---

## 6. Scale-Space Blob Detection Algorithm

### 6.1 Step 1 — Build the Scale-Space

Compute the **scale-normalised LoG** response at a discrete ladder of scales $\sigma_1 < \sigma_2 < \cdots < \sigma_n$, stacking the responses into a 3D scale-space volume $(x, y, \sigma)$.

### 6.2 Step 2 — Detect 3D Extrema

Locate local maxima and minima of the squared LoG response. Each candidate $(x, y, \sigma_k)$ is compared with its **26 neighbours**:

- $8$ spatial neighbours in the same scale layer.
- $9$ neighbours in the layer above ($\sigma_{k+1}$).
- $9$ neighbours in the layer below ($\sigma_{k-1}$).

A point that is strictly extremal among all 26 is declared a keypoint with image location $(x, y)$ and **characteristic scale** $\sigma^{*}$.

---

## 7. Efficient Implementation — Difference of Gaussians (DoG)

### 7.1 DoG Approximates the Scale-Normalised LoG

The LoG is computationally expensive to compute directly. From the heat-equation identity $\partial G_\sigma / \partial \sigma = \sigma\, \nabla^2 G_\sigma$, a first-order Taylor expansion in $\sigma$ gives

$$\mathrm{DoG}(x, y, \sigma) \;=\; G(x, y, k\sigma) - G(x, y, \sigma) \;\approx\; (k - 1)\, \sigma^2\, \nabla^2 g_\sigma.$$

Thus **DoG approximates the scale-normalised LoG up to the constant factor $(k - 1)$** — two Gaussian smoothings and a subtraction replace an explicit second derivative.

### 7.2 Which Scale Does a Detected Blob Belong To?

The expansion above approximates the LoG at the **smaller** scale $\sigma$ (not $k\sigma$). A DoG extremum detected between scales $\sigma$ and $k\sigma$ is therefore assigned scale $\sigma$, and the corresponding blob radius is $r = \sigma\sqrt{2}$.

### 7.3 DoG Pyramid Construction *(Lowe, IJCV 2004)*

1. Build a **Gaussian pyramid** — within each *octave*, blur the image at scales $\sigma, k\sigma, k^2\sigma, \ldots$
2. Subtract adjacent Gaussian layers to obtain a stack of **DoG images**.
3. Search the DoG stack for 3D scale-space extrema using the 26-neighbour test of §6.2.
4. Move to the next octave by downsampling by 2 — halving the resolution and doubling the effective scale.

The **absolute scale** of a keypoint at layer index $i$ within octave $o$ is

$$\sigma_{\mathrm{absolute}} \;=\; \sigma_0 \cdot k^{\,i} \cdot 2^{\,o}, \qquad k = 2^{1 / s},$$

where $s$ is the number of scale levels per octave and $\sigma_0$ is the base scale.

---

## 8. SIFT Descriptor

Once blobs are localised, each is described using **SIFT** (Scale-Invariant Feature Transform):

1. Take a **$16 \times 16$ pixel patch** centred on the keypoint.
2. Divide the patch into a **$4 \times 4$ grid** of cells ($4 \times 4$ pixels each).
3. Compute an **8-bin gradient orientation histogram** in each cell.
4. Concatenate the histograms into a single $4 \times 4 \times 8 = \mathbf{128}$-dimensional descriptor vector.

### 8.1 Invariance Properties

- **Rotation invariance.** Estimate the dominant gradient orientation in a region around the keypoint, then **rotate the patch to align with that orientation** before computing the descriptor. The same blob viewed under two rotations yields identical descriptors.
- **Illumination invariance.** **$L_2$-normalise** the 128-dimensional vector to unit length — this cancels multiplicative brightness changes. Additive intensity offsets are already cancelled because gradients depend only on local intensity *differences*.

---

## 9. Full Pipeline Summary

```
Image(s)
   │
   ▼
1. Blob Detection         Scale-normalised DoG pyramid at multiple σ
                          → 3D extrema in (x, y, σ) scale-space
   │
   ▼
2. Keypoint Description   128-dim SIFT descriptor per blob
                          → rotation- and illumination-normalised
   │
   ▼
3. Feature Matching       Nearest-neighbour search in descriptor space
   │
   ▼
4. Geometric Alignment    Homography / RANSAC for stitching, reconstruction
```