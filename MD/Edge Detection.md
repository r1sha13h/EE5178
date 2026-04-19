Here are the complete, comprehensive notes synthesizing everything from the entire chat history:

***

# Edge Detection — Comprehensive Notes
*EE5178: Modern Computer Vision | Slides 51–54 | Courtesy: Juan Carlos Niebles, Ranjay Krishna, Saad J. Bedros*

***

## Part 1: Biological Motivation

Hubel & Wiesel's experiments implanted electrodes into a cat's visual cortex and found that individual neurons fire **selectively for specific oriented edges**. This established that the biological visual system is literally a bank of oriented edge detectors — edges are the most primitive universal visual feature. [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/86324620/608f9762-0bcb-40c1-8a4e-f937c255ee8e/image.jpg)

***

## Part 2: What Is an Edge?

An **edge** is a location of **rapid, significant local intensity change** in an image. Physically caused by:
- Surface normal discontinuity (two surfaces meeting at an angle)
- Depth discontinuity (object occludes background)
- Surface colour/reflectance change (material boundary)
- Illumination discontinuity (cast shadow)

***

## Part 3: Mathematical Characterization

### 3.1 Three Equivalent Definitions of an Edge (1D view)

For a scanline intensity profile \(f(x)\):

1. Where \(f(x)\) changes most rapidly
2. Where \(|f'(x)|\) is a **local maximum**
3. Where \(f''(x) = 0\) — a **zero-crossing** of the second derivative

These give two algorithmic families:

| Family | Principle | Detectors |
|---|---|---|
| **Gradient-based** | Find peaks in \(|\nabla I|\) | Sobel, Canny |
| **Laplacian-based** | Find zero-crossings of \(\nabla^2 I\) | LoG (Marr-Hildreth) |

***

## Part 4: Discrete Derivatives — Full Derivations from Taylor Series

### 4.1 First Derivative: Forward Difference

Taylor expand \(f(x+h)\) about \(x\):

\[f(x+h) = f(x) + hf'(x) + \frac{h^2}{2!}f''(x) + \frac{h^3}{3!}f'''(x) + \cdots\]

Rearrange to isolate \(f'(x)\):

\[f'(x) = \frac{f(x+h) - f(x)}{h} - \frac{h}{2}f''(x) - \cdots\]

Set \(h=1\), drop \(O(h)\) truncation error:

\[\boxed{f'(x) \approx f(x+1) - f(x), \quad \text{mask: } [-1 \quad 1]}\]

**Accuracy:** First-order, \(O(h)\). Actually estimates the slope at \(x + \frac{1}{2}\), not at \(x\) — asymmetric.

### 4.2 First Derivative: Central Difference

Use both expansions simultaneously:

\[f(x+h) = f(x) + hf'(x) + \frac{h^2}{2}f''(x) + \frac{h^3}{6}f'''(x) + \cdots \quad \text{...(1)}\]

\[f(x-h) = f(x) - hf'(x) + \frac{h^2}{2}f''(x) - \frac{h^3}{6}f'''(x) + \cdots \quad \text{...(2)}\]

Subtract (2) from (1) — all **even-order terms cancel**:

\[f(x+h) - f(x-h) = 2hf'(x) + \frac{2h^3}{6}f'''(x) + \cdots\]

Set \(h=1\), drop \(O(h^2)\) error:

\[\boxed{f'(x) \approx \frac{f(x+1) - f(x-1)}{2}, \quad \text{mask: } [-1 \quad 0 \quad 1]}\]

**Accuracy:** Second-order, \(O(h^2)\). Centered exactly at \(x\) — the standard choice for edge detection.

### 4.3 Second Derivative: Central (Symmetric) Derivation

**Add** expansions (1) and (2) instead — odd-order terms cancel:

\[f(x+h) + f(x-h) = 2f(x) + h^2f''(x) + \frac{h^4}{12}f^{(4)}(x) + \cdots\]

Rearrange, set \(h=1\), drop \(O(h^2)\):

\[\boxed{f''(x) \approx f(x+1) - 2f(x) + f(x-1), \quad \text{mask: } [1 \quad -2 \quad 1]}\]

**Accuracy:** Second-order, \(O(h^2)\). Correctly centered at \(x\).

> **Important note:** The notes previously showed a derivation via \(f''(x) \approx f'(x+1) - f'(x)\) which gives \(f(x+2) - 2f(x+1) + f(x)\) — this is centered at \(x+1\), not \(x\). A re-centering substitution \(x \leftarrow x-1\) is required to recover the formula above. The Taylor series approach is cleaner and avoids this issue entirely.

***

## Part 5: The Noise Problem

### 5.1 Why Derivatives Amplify Noise

For \(I(x) = \hat{I}(x) + N(x)\) where \(N(x) \sim \mathcal{N}(0,\sigma^2)\) i.i.d.:

The central difference noise term: \(N_d(x) = N(x+1) - N(x-1)\)

\[E[N_d] = 0 \quad \text{(unbiased)}\]
\[\text{Var}(N_d) = (1)^2\sigma^2 + (-1)^2\sigma^2 = 2\sigma^2\]

The 1st derivative **doubles** the noise variance. The general rule for any linear filter \(\{h_k\}\):

\[\text{Var}(N_{\text{output}}) = \sigma^2 \sum_k h_k^2\]

| Operator | Mask | \(\sum h_k^2\) | Noise Variance |
|---|---|---|---|
| Image | — | 1 | \(\sigma^2\) |
| 1st derivative | \([-1,0,1]\) | 2 | \(2\sigma^2\) |
| 2nd derivative (1D) | \([1,-2,1]\) | 6 | \(6\sigma^2\) |
| 2D Laplacian | \([0,1,0;1,-4,1;0,1,0]\) | 20 | \(20\sigma^2\) |

### 5.2 The Solution: Smooth Before Differentiating

Apply Gaussian blur first. The 1D kernel \([1,2,1]/4\) comes from evaluating \(e^{-x^2/2}\) at \(x\in\{-1,0,1\}\):

\[e^{-1/2} \approx 0.607,\quad e^0 = 1.0 \quad \Rightarrow \quad \text{unnormalized: }[0.607, 1.0, 0.607] \approx [1,2,1]\]

This is also the 2nd row of Pascal's triangle — repeated convolution of \([1,1]\) with itself converges to a Gaussian by the Central Limit Theorem.

### 5.3 Scale–Noise Tradeoff

| \(\sigma\) | Noise suppression | Edge localization | Detects |
|---|---|---|---|
| Small | Weak | Sharp, precise | Fine details |
| Large | Strong | Blurred/spread | Coarse edges only |

There is no universally optimal \(\sigma\) — it is application-dependent.

***

## Part 6: Derivative Theorem of Convolution (Key Identity)

### 6.1 The Identity

\[\frac{d}{dx}(f * G) = f * \frac{dG}{dx}\]

In 2D:

\[\nabla^2[f * G] = f * \nabla^2 G\]

### 6.2 Proof

Write convolution explicitly:

\[(f*G)(x) = \int_{-\infty}^{\infty} f(u)\,G(x-u)\,du\]

Differentiate w.r.t. \(x\) — since \(f(u)\) does not depend on \(x\), the derivative passes inside the integral (Leibniz rule):

\[\frac{d}{dx}(f*G) = \int f(u)\,\frac{d}{dx}G(x-u)\,du = f * \frac{dG}{dx}\]

The same applies to \(\nabla^2\) since it is a linear differential operator.

### 6.3 Practical Consequence

Without the identity: two sequential operations — smooth the image, then differentiate.

With the identity: **precompute** the Derivative of Gaussian kernel \(\frac{dG}{dx}\) once offline, then apply as a **single convolution** at runtime. Cheaper and cleaner.

***

## Part 7: The Three Criteria for an Optimal Edge Detector

Formally stated in slides 51–54: [ppl-ai-file-upload.s3.amazonaws](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/images/86324620/608f9762-0bcb-40c1-8a4e-f937c255ee8e/image.jpg)

| Criterion | Meaning |
|---|---|
| **Good detection** | Minimize false positives (spurious noise edges) AND false negatives (missed real edges) |
| **Good localization** | Detected edge as close as possible to true edge location |
| **Single response** | One output pixel per true edge — minimize local maxima around the edge |

The visual illustration in the slides shows four cases: True edge (red), Poor robustness to noise (purple — scattered detections), Poor localization (green — offset detections), Too many responses (black — multiple detections per edge).

***

## Part 8: Sobel Edge Detector

### 8.1 Kernels

\[G_x = \begin{bmatrix}+1&0&-1\\+2&0&-2\\+1&0&-1\end{bmatrix}, \qquad G_y = \begin{bmatrix}+1&+2&+1\\0&0&0\\-1&-2&-1\end{bmatrix}\]

### 8.2 Separability

\[G_x = \begin{bmatrix}1\\2\\1\end{bmatrix} \cdot \begin{bmatrix}+1 & 0 & -1\end{bmatrix}\]

The \([1,2,1]^T\) factor is the discrete Gaussian (smoothing perpendicular to the derivative direction); \([+1,0,-1]\) is the central difference derivative. Sobel simultaneously smooths and differentiates.

### 8.3 Output

\[G = \sqrt{G_x^2 + G_y^2}, \qquad \Theta = \text{atan2}(G_y, G_x)\]

Single threshold \(\tau\): if \(G > \tau\), declare edge.

### 8.4 Limitations

- Only mild noise suppression from \([1,2,1]\) — not a true Gaussian
- **Thick edges** — no thinning; gradient ridge spans multiple pixels
- Single threshold — brittle, breaks contours at weak points
- No edge connectivity
- Directional bias toward horizontal/vertical edges

***

## Part 9: Canny Edge Detector

### 9.1 Step 1 — Derivatives of Gaussian (DoG)

Using the key identity:

\[I_x = I * \frac{\partial G_\sigma}{\partial x}, \qquad I_y = I * \frac{\partial G_\sigma}{\partial y}\]

Single convolution = smooth + differentiate simultaneously. Noise variance: \(2\sigma^2\) (same as 1st derivative, but the Gaussian controls how much noise entered in the first place).

### 9.2 Step 2 — Gradient Magnitude and Direction

\[\|\nabla I\| = \sqrt{I_x^2 + I_y^2}, \qquad \theta = \text{atan2}(I_y, I_x)\]

After this step: thick gradient ridges — violates the single-response criterion.

### 9.3 Step 3 — Non-Maximum Suppression (NMS)

**Goal:** Thin ridges to single-pixel-wide edges.

**Rule:** Keep pixel \(q\) at \((x,y)\) iff its gradient magnitude is **strictly greater** than both neighbors along gradient direction \(\theta\):

\[\|\nabla I(q)\| > \|\nabla I(p)\| \;\text{ AND }\; \|\nabla I(q)\| > \|\nabla I(r)\| \implies q\text{ is edge}\]

Otherwise suppress. The continuous angle \(\theta\) is quantized to the nearest of 4 axes (0°, 45°, 90°, 135°):

| Gradient Angle Range | Quantized to | Neighbors Compared |
|---|---|---|
| \(-22.5°\) to \(22.5°\) | 0° | Left & Right |
| \(22.5°\) to \(67.5°\) | 45° | Top-right & Bottom-left |
| \(67.5°\) to \(112.5°\) | 90° | Top & Bottom |
| \(112.5°\) to \(157.5°\) | 135° | Top-left & Bottom-right |

Sub-pixel interpolation gives more accurate neighbor gradient values between grid points.

### 9.4 Step 4 — Hysteresis Thresholding

A single threshold breaks contours at weak points. Dual threshold + connectivity solves this:

\[\|\nabla I\| \geq T_{\text{high}} \implies \text{strong edge (always kept)}\]
\[T_{\text{low}} \leq \|\nabla I\| < T_{\text{high}} \implies \text{weak edge (keep only if connected to strong)}\]
\[\|\nabla I\| < T_{\text{low}} \implies \text{discard}\]

Weak edges connected (directly or transitively) to a strong edge are retained — this traces contours through intensity dips without breaking them.

### 9.5 Full Canny Pipeline

```
Input Image I
      │
      ▼
① I_x = I * ∂G/∂x,   I_y = I * ∂G/∂y       (smooth + diff in one step)
      │
      ▼
② |∇I| = √(Ix²+Iy²),  θ = atan2(Iy, Ix)     (gradient magnitude + direction)
      │
      ▼
③ Non-Maximum Suppression                      (thin ridges → 1-pixel wide)
      │
      ▼
④ Hysteresis Thresholding (T_low, T_high)      (link weak→strong edges)
      │
      ▼
Final single-pixel edge map
```

***

## Part 10: Laplacian and 2D Second-Order Operators

### 10.1 2D Laplacian Mask Derivation

Add the two 1D second-derivative masks along each axis:

\[\underbrace{\begin{bmatrix}0&0&0\\1&-2&1\\0&0&0\end{bmatrix}}_{\partial^2/\partial x^2} + \underbrace{\begin{bmatrix}0&1&0\\0&-2&0\\0&1&0\end{bmatrix}}_{\partial^2/\partial y^2} = \underbrace{\begin{bmatrix}0&1&0\\1&-4&1\\0&1&0\end{bmatrix}}_{\nabla^2}\]

8-connected version (includes diagonals): center = −8, all 8 neighbors = +1.

**Key properties:**
- Coefficients sum to **zero** → flat regions give zero response
- **Isotropic** — equal response in all directions
- **No directional information** — cannot determine edge orientation
- Very noise-sensitive: \(20\sigma^2\) amplification in 2D — practically unusable on raw images

***

## Part 11: Laplacian of Gaussian (LoG)

### 11.1 Key Identity (Proof)

\[\nabla^2[f * G_\sigma] = f * \nabla^2 G_\sigma\]

Write convolution:

\[\nabla^2[f*G] = \nabla^2 \int f(u,v)\,G(x-u, y-v)\,du\,dv\]

Since \(f(u,v)\) does not depend on \((x,y)\), \(\nabla^2\) passes inside the integral:

\[= \int f(u,v)\,\nabla^2 G(x-u,y-v)\,du\,dv = f * \nabla^2 G \quad \square\]

### 11.2 The LoG Kernel Analytically

\[\text{LoG}(x,y) = \frac{-1}{\pi\sigma^4}\left(1 - \frac{x^2+y^2}{2\sigma^2}\right)e^{-\frac{x^2+y^2}{2\sigma^2}}\]

Three factors:
- **Prefactor** \(\frac{-1}{\pi\sigma^4}\): negative scaling
- **Polynomial** \(\left(1 - \frac{r^2}{2\sigma^2}\right)\): changes sign at \(r = \sigma\sqrt{2}\)
- **Gaussian** \(e^{-r^2/2\sigma^2}\): always positive, ensures spatial decay

**Mexican hat / sombrero shape:**
- Negative at center where \(r < \sigma\sqrt{2}\)
- Zero at \(r = \sigma\sqrt{2}\) — **this is the edge location**
- Positive in surrounding ring where \(r > \sigma\sqrt{2}\)

**Mandatory constraint:** All kernel values must sum to **exactly zero**.

***

## Part 12: LoG Kernel Derivations for Three Sizes

### Kernel 1: \(\sigma = 1.0\), \(3 \times 3\)

Prefactor: \(\frac{-1}{\pi}\). Evaluate at \((x,y) \in \{-1,0,+1\}^2\). Note: corners have \(r^2 = 2 = 2\sigma^2\), so they fall exactly on the zero-crossing ring:

\[K_{\sigma=1} = \begin{bmatrix}+4 & -1 & +4\\-1 & -13 & -1\\+4 & -1 & +4\end{bmatrix}\]

Zero-crossing at \(r = 1 \cdot \sqrt{2} \approx 1.41\) px ✓

### Kernel 2: \(\sigma = \sqrt{2} \approx 1.4\), \(5 \times 5\)

Prefactor: \(\frac{-1}{4\pi}\). The axis-distance-2 positions have \(r^2 = 4 = 2\sigma^2 = 4\) — again exactly on the zero-crossing ring, giving \(\text{LoG} = 0\):

\[K_{\sigma=\sqrt{2}} = \begin{bmatrix}0 & 0 & -1 & 0 & 0\\0 & -1 & -2 & -1 & 0\\-1 & -2 & 16 & -2 & -1\\0 & -1 & -2 & -1 & 0\\0 & 0 & -1 & 0 & 0\end{bmatrix}\]

This is the **standard 5×5 LoG kernel** in literature. Zero-crossing at \(r = \sqrt{2}\cdot\sqrt{2} = 2.0\) px ✓

### Kernel 3: \(\sigma = 2.0\), \(7 \times 7\)

Prefactor: \(\frac{-1}{16\pi}\). Zero-crossing at \(r = 2\sqrt{2} \approx 2.83\) px:

\[K_{\sigma=2} \approx \frac{1}{1000}\begin{bmatrix}+5&+5&+4&+3&+4&+5&+5\\+5&+2&-1&-3&-1&+2&+5\\+4&-1&-8&-11&-8&-1&+4\\+3&-3&-11&-15&-11&-3&+3\\+4&-1&-8&-11&-8&-1&+4\\+5&+2&-1&-3&-1&+2&+5\\+5&+5&+4&+3&+4&+5&+5\end{bmatrix}\]

Sum ≈ 0 ✓

### Summary: LoG Kernel Properties

| | \(\sigma=1.0\), 3×3 | \(\sigma=\sqrt{2}\), 5×5 | \(\sigma=2.0\), 7×7 |
|---|---|---|---|
| **Prefactor** | \(-1/\pi\) | \(-1/4\pi\) | \(-1/16\pi\) |
| **Zero-crossing \(r\)** | 1.41 px | 2.00 px | 2.83 px |
| **Detects edges at** | Fine scale | Standard | Coarse scale |
| **Noise sensitivity** | High | Moderate | Low |

***

## Part 13: LoG Thresholding — Complete Procedure

### Why NOT threshold \(|R| > \tau\) directly?

LoG oscillates ±. Thresholding the response magnitude detects the ridges on both sides of the edge — producing **two parallel lines per edge**, not the edge itself.

### Stage 1: Detect Zero-Crossings

Compute \(R = f * \nabla^2 G_\sigma\). Pixel \((i,j)\) is a candidate edge if:

\[R(i,j) > 0 \;\text{ and }\; R(i,j+1) < 0 \quad \text{(or any adjacent pair sign change)}\]

Strength of zero-crossing:
\[\text{strength} = |R(p) - R(q)|\]
where \(p, q\) straddle the crossing. Large difference = sharp, confident edge.

### Stage 2: Gradient-Magnitude Gating

\[\text{Keep zero-crossing at }(x,y) \iff |\nabla I|(x,y) > \tau\]

**Intuition:** True edge = zero-crossing in \(\nabla^2 I\) **AND** large \(|\nabla I|\) on either side. Noise fluctuation = zero-crossing but tiny \(|\nabla I|\).

### Full LoG Pipeline

```
I(x,y)
   │
   ▼
R = I * ∇²G_σ              (LoG in single convolution)
   │
   ▼
Detect sign changes in R   (candidate zero-crossings)
   │
   ▼
Compute |∇I| of smoothed image
   │
   ▼
Keep only where |∇I| > τ   (final edge map)
```

***

## Part 14: Difference of Gaussians (DoG)

\[\text{DoG}(x,y) = G_{\sigma_1}(x,y) - G_{\sigma_2}(x,y), \qquad \sigma_1 < \sigma_2\]

**Why it approximates LoG:**

\[\frac{\partial G_\sigma}{\partial \sigma} \propto \nabla^2 G_\sigma\]

So the finite difference \(G_{\sigma+\Delta\sigma} - G_\sigma \approx \Delta\sigma \cdot \frac{\partial G}{\partial \sigma} \propto \nabla^2 G_\sigma\). DoG is cheaper (two blurs, no explicit second derivative) and forms the basis of **SIFT** for multi-scale keypoint detection.

***

## Part 15: Full Detector Comparison

| Property | Sobel | Canny | LoG (Marr-Hildreth) |
|---|---|---|---|
| **Derivative order** | 1st | 1st | 2nd |
| **Noise handling** | Mild \([1,2,1]\) | Explicit \(G_\sigma\) | \(G_\sigma\) embedded in LoG |
| **Noise amplification** | \(2\sigma^2\) | \(2\sigma^2\) | \(6\sigma^2\) (1D), \(20\sigma^2\) (2D) |
| **Edge thickness** | Thick (multi-px) | 1-pixel (NMS) | 1-pixel (zero-crossing) |
| **Thresholding** | Single \(\tau\) on \(G\) | Dual hysteresis \((T_\text{lo}, T_\text{hi})\) | Zero-crossing + \(|\nabla I|>\tau\) |
| **Edge connectivity** | None | Yes (hysteresis) | Moderate |
| **Directional info** | Yes \((\theta)\) | Yes \((\theta)\) | No |
| **Corner handling** | Good | Good | Poor (rounds corners) |
| **Scale parameter** | None | \(\sigma\) | \(\sigma\) |
| **Best for** | Fast, clean images | General purpose (optimal) | Non-sharp edges, SIFT features |

***

## Part 16: Conceptual Map

```
Biological motivation (Hubel & Wiesel — oriented edge neurons)
          │
          ▼
Edges = rapid intensity change
          │
          ├─── 1st derivative → peak in |f'(x)|
          │         │
          │         ├── Finite diff: f(x+1)-f(x-1) → noise ×2σ²
          │         ├── SOLUTION: Smooth first (Gaussian [1,2,1])
          │         ├── KEY IDENTITY: d/dx(f*G) = f*(dG/dx)
          │         │
          │         ├── SOBEL: [1,2,1]ᵀ×[+1,0,-1] → G → single τ
          │         │         ✗ thick edges, no connectivity, no σ
          │         │
          │         └── CANNY: DoG → |∇I|,θ → NMS → hysteresis(T_lo,T_hi)
          │                   ✓ good detection + localization + single response
          │
          └─── 2nd derivative → zero-crossing of f''(x)
                    │
                    ├── [1,-2,1] → noise ×6σ² (1D), ×20σ² (2D)
                    ├── 2D Laplacian [0,1,0;1,-4,1;0,1,0]
                    ├── Must smooth: ∇²[f*G] = f*∇²G [KEY IDENTITY]
                    │
                    └── LOG: f*∇²G_σ → Mexican hat kernel
                              → zero-crossings (sign changes)
                              → gate with |∇I| > τ
                              → DoG ≈ LoG (cheaper) → used in SIFT
```