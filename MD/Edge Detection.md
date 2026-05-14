# Edge Detection

*EE5178 — Modern Computer Vision*

---

## 1. Biological Motivation

Hubel & Wiesel's experiments implanted electrodes into a cat's visual cortex and found that individual neurons fire **selectively for specific oriented edges**. This established that the biological visual system is, at its earliest stage, a *bank of oriented edge detectors* — edges are the most primitive universal visual feature.

---

## 2. What Is an Edge?

An **edge** is a location of **rapid, significant local intensity change** in an image. Physically, an edge can be caused by:

- **Surface-normal discontinuity** — two surfaces meeting at an angle.
- **Depth discontinuity** — an object occluding the background.
- **Surface colour / reflectance change** — a material boundary.
- **Illumination discontinuity** — a cast shadow.

---

## 3. Mathematical Characterization

### 3.1 Three Equivalent Definitions of an Edge (1D view)

For a scanline intensity profile $f(x)$, an edge is a location:

1. where $f(x)$ changes most rapidly,
2. where $|f'(x)|$ is a **local maximum**, or
3. where $f''(x) = 0$ — a **zero-crossing** of the second derivative.

These give two algorithmic families:

| Family | Principle | Detectors |
|---|---|---|
| **Gradient-based** | Find peaks in $\lvert \nabla I \rvert$ | Sobel, Canny |
| **Laplacian-based** | Find zero-crossings of $\nabla^2 I$ | LoG (Marr–Hildreth) |

---

## 4. Discrete Derivatives — Derivations from the Taylor Series

### 4.1 First Derivative — Forward Difference

Taylor-expand $f(x+h)$ about $x$:

$$f(x+h) = f(x) + h f'(x) + \frac{h^2}{2!} f''(x) + \frac{h^3}{3!} f'''(x) + \cdots$$

Rearrange to isolate $f'(x)$:

$$f'(x) = \frac{f(x+h) - f(x)}{h} - \frac{h}{2} f''(x) - \cdots$$

Set $h = 1$ and drop the $O(h)$ truncation error:

$$\boxed{f'(x) \approx f(x+1) - f(x), \qquad \text{mask: } [-1 \quad 1]}$$

**Accuracy.** First-order, $O(h)$. Strictly speaking it estimates the slope at $x + \tfrac{1}{2}$, not at $x$ — asymmetric.

### 4.2 First Derivative — Central Difference

Use both expansions simultaneously:

$$f(x+h) = f(x) + h f'(x) + \frac{h^2}{2} f''(x) + \frac{h^3}{6} f'''(x) + \cdots \tag{1}$$

$$f(x-h) = f(x) - h f'(x) + \frac{h^2}{2} f''(x) - \frac{h^3}{6} f'''(x) + \cdots \tag{2}$$

Subtract (2) from (1) — all **even-order terms cancel**:

$$f(x+h) - f(x-h) = 2 h f'(x) + \frac{2 h^3}{6} f'''(x) + \cdots$$

Set $h = 1$ and drop the $O(h^2)$ error:

$$\boxed{f'(x) \approx \frac{f(x+1) - f(x-1)}{2}, \qquad \text{mask: } [-1 \quad 0 \quad 1]}$$

**Accuracy.** Second-order, $O(h^2)$, centred exactly at $x$. This is the standard choice for edge detection.

### 4.3 Second Derivative — Central (Symmetric) Form

**Add** expansions (1) and (2) instead — odd-order terms cancel:

$$f(x+h) + f(x-h) = 2 f(x) + h^2 f''(x) + \frac{h^4}{12} f^{(4)}(x) + \cdots$$

Rearrange, set $h = 1$, drop the $O(h^2)$ error:

$$\boxed{f''(x) \approx f(x+1) - 2 f(x) + f(x-1), \qquad \text{mask: } [1 \quad -2 \quad 1]}$$

**Accuracy.** Second-order, $O(h^2)$, correctly centred at $x$.

> **Remark.** A common shortcut derivation uses $f''(x) \approx f'(x+1) - f'(x)$, which gives $f(x+2) - 2 f(x+1) + f(x)$ — centred at $x+1$, *not* $x$. A re-centering substitution $x \leftarrow x - 1$ is required to recover the symmetric formula above. The Taylor-series approach avoids this issue entirely.

---

## 5. The Noise Problem

### 5.1 Why Derivatives Amplify Noise

Model the observed scanline as the sum of a clean signal and additive noise:

$$I(x) = \hat{I}(x) + N(x), \qquad N(x) \sim \mathcal{N}(0, \sigma^2) \;\text{ i.i.d.}$$

Here $\hat{I}(x)$ is the (unknown) noise-free intensity and the noise samples $N(x)$ are independent and identically distributed Gaussians with zero mean and variance $\sigma^2$ — meaning $N(x_1)$ and $N(x_2)$ are uncorrelated whenever $x_1 \neq x_2$.

Now apply the central-difference operator from §4.2. Because the operator is **linear**, its action distributes over the sum:

$$I(x+1) - I(x-1) = \underbrace{\bigl[\hat{I}(x+1) - \hat{I}(x-1)\bigr]}_{\text{true signal derivative (}\,\approx 2 \hat{I}'(x)\,)} \;+\; \underbrace{\bigl[N(x+1) - N(x-1)\bigr]}_{\text{noise the operator picked up}}.$$

The first bracket is exactly what we *want* — twice the central-difference estimate of $\hat{I}'(x)$. The second bracket is the unavoidable noise contribution introduced by differentiating a noisy signal. Isolating it and giving it a name,

$$N_d(x) \;\triangleq\; N(x+1) - N(x-1),$$

we can study how much noise the derivative filter amplifies, independent of the underlying scene.

Its statistics are

$$E[N_d] = 0 \quad \text{(unbiased)}, \qquad \mathrm{Var}(N_d) = (1)^2 \sigma^2 + (-1)^2 \sigma^2 = 2 \sigma^2.$$

The first derivative **doubles** the noise variance. The general rule for any linear filter $\{h_k\}$ applied to i.i.d. noise is

$$\mathrm{Var}(N_{\text{output}}) = \sigma^2 \sum_k h_k^2.$$

| Operator | Mask | $\sum_k h_k^2$ | Output Noise Variance |
|---|---|---|---|
| Image (no filter) | — | 1 | $\sigma^2$ |
| 1st derivative | $[-1, 0, 1]$ | 2 | $2 \sigma^2$ |
| 2nd derivative (1D) | $[1, -2, 1]$ | 6 | $6 \sigma^2$ |
| 2D Laplacian | $\bigl[\begin{smallmatrix}0&1&0\\1&-4&1\\0&1&0\end{smallmatrix}\bigr]$ | 20 | $20 \sigma^2$ |

#### Worked Example — 2D Laplacian

Repeat the same exercise in two dimensions. Model the noisy image as

$$I(x, y) = \hat{I}(x, y) + N(x, y), \qquad N(x, y) \sim \mathcal{N}(0, \sigma^2) \;\text{ i.i.d. across pixels.}$$

Apply the 4-connected discrete Laplacian:

$$\nabla^2 I(x, y) \;\approx\; I(x+1, y) + I(x-1, y) + I(x, y+1) + I(x, y-1) - 4\, I(x, y).$$

By linearity, the operator splits cleanly into a signal piece and a noise piece:

$$\nabla^2 I(x, y) = \underbrace{\nabla^2 \hat{I}(x, y)}_{\text{true Laplacian of clean image}} \;+\; \underbrace{N_L(x, y)}_{\text{operator-induced noise}},$$

where

$$N_L(x, y) \;\triangleq\; N(x+1, y) + N(x-1, y) + N(x, y+1) + N(x, y-1) - 4\, N(x, y).$$

**Mean.** All five noise samples have zero mean, so by linearity of expectation,

$$E[N_L] = 0 \quad \text{(unbiased)}.$$

**Variance.** The five samples are evaluated at distinct pixel locations and are therefore mutually independent. The variance of a linear combination of independent random variables is the sum of (coefficient$^2 \times$ variance):

$$\mathrm{Var}(N_L) = (1)^2 \sigma^2 + (1)^2 \sigma^2 + (1)^2 \sigma^2 + (1)^2 \sigma^2 + (-4)^2 \sigma^2 = (1 + 1 + 1 + 1 + 16)\, \sigma^2 = 20\, \sigma^2.$$

This matches the $\sum_k h_k^2 = 20$ row of the table. The 2D Laplacian amplifies the noise variance by a factor of **20** — almost an order of magnitude worse than a single first derivative. In standard-deviation units the noise grows by $\sqrt{20} \approx 4.47\times$, which is why the raw Laplacian is essentially unusable on real images and must be paired with smoothing (the LoG of §11).

### 5.2 The Solution — Smooth Before Differentiating

Apply a Gaussian blur first. The 1D smoothing kernel $[1, 2, 1] / 4$ comes from evaluating $e^{-x^2 / 2}$ at $x \in \{-1, 0, +1\}$:

$$e^{-1/2} \approx 0.607, \qquad e^{0} = 1.0 \quad \Longrightarrow \quad \text{unnormalized: } [0.607,\, 1.0,\, 0.607] \approx [1, 2, 1].$$

This is also the second row of Pascal's triangle — repeated convolution of $[1, 1]$ with itself converges to a Gaussian by the Central Limit Theorem.

**Proof that smoothing reduces noise.** Apply $h = \tfrac{1}{4}[1, 2, 1]$ to the noisy signal $I(x) = \hat{I}(x) + N(x)$. By linearity,

$$(h * I)(x) = \underbrace{\tfrac{1}{4}\bigl[\hat{I}(x-1) + 2\hat{I}(x) + \hat{I}(x+1)\bigr]}_{\text{smoothed signal}} \;+\; \underbrace{\tfrac{1}{4}\bigl[N(x-1) + 2 N(x) + N(x+1)\bigr]}_{\;N_s(x)\,:\, \text{smoothed noise}}.$$

The three noise samples are i.i.d., so by the rule $\mathrm{Var}(\sum a_k X_k) = \sum a_k^2 \mathrm{Var}(X_k)$,

$$E[N_s] = 0, \qquad \mathrm{Var}(N_s) = \tfrac{1}{16}\bigl((1)^2 + (2)^2 + (1)^2\bigr)\, \sigma^2 = \tfrac{6}{16}\, \sigma^2 = \tfrac{3}{8}\, \sigma^2.$$

The noise variance drops from $\sigma^2$ to $\tfrac{3}{8}\sigma^2 = 0.375\, \sigma^2$ — a $\tfrac{8}{3} \approx 2.67\times$ reduction (or $\sqrt{8/3} \approx 1.63\times$ in standard deviation). Intuitively, averaging three independent samples lets their zero-mean errors partially cancel, while the signal — locally similar across the three taps — is preserved.

### 5.3 Scale–Noise Trade-off

| $\sigma$ | Noise suppression | Edge localization | Detects |
|---|---|---|---|
| Small | Weak | Sharp, precise | Fine details |
| Large | Strong | Blurred / spread | Coarse edges only |

There is no universally optimal $\sigma$ — it is application-dependent.

---

## 6. Derivative Theorem of Convolution (Key Identity)

### 6.1 The Identity

$$\frac{d}{dx}(f * G) = f * \frac{dG}{dx}.$$

In 2D:

$$\nabla^2 [f * G] = f * \nabla^2 G.$$

### 6.2 Proof

Write convolution explicitly:

$$(f * G)(x) = \int_{-\infty}^{\infty} f(u)\, G(x - u)\, du.$$

Differentiate with respect to $x$. Since $f(u)$ does not depend on $x$, the derivative passes inside the integral (Leibniz's rule):

$$\frac{d}{dx}(f * G) = \int f(u)\, \frac{d}{dx} G(x - u)\, du = f * \frac{dG}{dx}.$$

The same argument applies to $\nabla^2$ since it is a linear differential operator. $\square$

### 6.3 Practical Consequence

Without the identity, edge detection requires **two sequential convolutions** at runtime — first smooth the image with the Gaussian $G$, then differentiate the smoothed result.

With the identity, we can **precompute** the Derivative-of-Gaussian kernel $\tfrac{dG}{dx}$ once offline and apply it as a **single convolution** at runtime — cheaper and cleaner.

---

## 7. Three Criteria for an Optimal Edge Detector

| Criterion | Meaning |
|---|---|
| **Good detection** | Minimize false positives (spurious noise edges) *and* false negatives (missed real edges). |
| **Good localization** | Detected edge as close as possible to the true edge location. |
| **Single response** | One output pixel per true edge — minimize local maxima around the edge. |

The visual illustration in the slides shows four cases: True edge (red), poor robustness to noise (purple — scattered detections), poor localization (green — offset detections), and too many responses (black — multiple detections per edge).

---

## 8. Sobel Edge Detector

### 8.1 Kernels

$$G_x = \begin{bmatrix}+1 & 0 & -1 \\ +2 & 0 & -2 \\ +1 & 0 & -1\end{bmatrix}, \qquad G_y = \begin{bmatrix}+1 & +2 & +1 \\ 0 & 0 & 0 \\ -1 & -2 & -1\end{bmatrix}.$$

### 8.2 Separability

$$G_x = \begin{bmatrix}1 \\ 2 \\ 1\end{bmatrix} \cdot \begin{bmatrix}+1 & 0 & -1\end{bmatrix}.$$

The factor $[1, 2, 1]^T$ is the discrete Gaussian (smoothing perpendicular to the derivative direction); $[+1, 0, -1]$ is the central-difference derivative. Sobel **simultaneously smooths and differentiates** in one separable convolution.

### 8.3 Output

$$G = \sqrt{G_x^2 + G_y^2}, \qquad \Theta = \mathrm{atan2}(G_y, G_x).$$

A single threshold $\tau$: if $G > \tau$, declare an edge.

### 8.4 Limitations

- Only mild noise suppression from $[1, 2, 1]$ — not a true Gaussian.
- **Thick edges** — no thinning; the gradient ridge spans multiple pixels.
- **Single threshold is brittle.** Gradient magnitude varies along a real contour, so any fixed $\tau$ either drops weak segments (fragmenting the edge) or admits noise — fixed by Canny's hysteresis (§9.4).
- **No edge connectivity.** Each pixel is thresholded independently, producing a set of points rather than coherent curves; linking requires a separate post-process.
- **Directional bias.** The $[1, 2, 1]^T$ smoothing weight is densest along the axes and weakest at the corners, so 45° edges get a systematically weaker response and an axis-biased gradient direction.

---

## 9. Canny Edge Detector

### 9.1 Step 1 — Derivatives of Gaussian (DoG)

Using the key identity from Section 6:

$$I_x = I * \frac{\partial G_\sigma}{\partial x}, \qquad I_y = I * \frac{\partial G_\sigma}{\partial y}.$$

A single convolution simultaneously smooths and differentiates. The output noise variance is $2 \sigma^2$ (same form as the first derivative), but the Gaussian controls *how much* noise enters in the first place.

### 9.2 Step 2 — Gradient Magnitude and Direction

$$\|\nabla I\| = \sqrt{I_x^2 + I_y^2}, \qquad \theta = \mathrm{atan2}(I_y, I_x).$$

After this step, the gradient ridges are still *thick* — the single-response criterion is violated.

### 9.3 Step 3 — Non-Maximum Suppression (NMS)

**Goal.** Thin ridges to single-pixel-wide edges.

**Rule.** Keep pixel $q$ at $(x, y)$ if and only if its gradient magnitude is **strictly greater** than the magnitudes at both neighbours along the gradient direction $\theta$:

$$\|\nabla I(q)\| > \|\nabla I(p)\| \;\text{ and }\; \|\nabla I(q)\| > \|\nabla I(r)\| \;\Longrightarrow\; q \text{ is an edge}.$$

Otherwise suppress. The continuous angle $\theta$ is quantized to the nearest of four axes ($0°, 45°, 90°, 135°$):

| Gradient angle range | Quantized to | Neighbours compared |
|---|---|---|
| $-22.5°$ to $22.5°$ | $0°$ | Left & Right |
| $22.5°$ to $67.5°$ | $45°$ | Top-right & Bottom-left |
| $67.5°$ to $112.5°$ | $90°$ | Top & Bottom |
| $112.5°$ to $157.5°$ | $135°$ | Top-left & Bottom-right |

Sub-pixel interpolation gives more accurate neighbour gradient values between grid points.

### 9.4 Step 4 — Hysteresis Thresholding

A single threshold breaks contours at weak points. A dual threshold combined with connectivity solves this:

$$\|\nabla I\| \geq T_{\text{high}} \;\Longrightarrow\; \text{strong edge (always kept)},$$

$$T_{\text{low}} \leq \|\nabla I\| < T_{\text{high}} \;\Longrightarrow\; \text{weak edge (kept only if connected to a strong edge)},$$

$$\|\nabla I\| < T_{\text{low}} \;\Longrightarrow\; \text{discard}.$$

Weak edges connected — directly or transitively — to a strong edge are retained. This traces contours through intensity dips without breaking them.

### 9.5 Full Canny Pipeline

```
Input Image I
      │
      ▼
① I_x = I * ∂G/∂x,   I_y = I * ∂G/∂y       (smooth + differentiate in one step)
      │
      ▼
② |∇I| = √(Ix²+Iy²),  θ = atan2(Iy, Ix)     (gradient magnitude + direction)
      │
      ▼
③ Non-Maximum Suppression                     (thin ridges → 1-pixel wide)
      │
      ▼
④ Hysteresis Thresholding (T_low, T_high)     (link weak → strong edges)
      │
      ▼
Final single-pixel edge map
```

---

## 10. The Laplacian and 2D Second-Order Operators

### 10.1 Definition — Divergence of the Gradient

The Laplacian is the dot product of the gradient operator with itself:

$$\nabla^2 \;=\; \nabla \cdot \nabla \;=\; \begin{bmatrix}\partial / \partial x\\ \partial / \partial y\end{bmatrix} \cdot \begin{bmatrix}\partial / \partial x\\ \partial / \partial y\end{bmatrix} \;=\; \frac{\partial^2}{\partial x^2} + \frac{\partial^2}{\partial y^2}.$$

Applied to an image:

$$\nabla^2 I(x, y) \;=\; \frac{\partial^2 I}{\partial x^2} + \frac{\partial^2 I}{\partial y^2}.$$

In discrete form, using central second differences along each axis,

$$\frac{\partial^2 I}{\partial x^2} \approx I(i, j+1) - 2 I(i, j) + I(i, j-1), \qquad \frac{\partial^2 I}{\partial y^2} \approx I(i+1, j) - 2 I(i, j) + I(i-1, j),$$

so

$$\nabla^2 I(i, j) \;\approx\; I(i, j+1) + I(i, j-1) + I(i+1, j) + I(i-1, j) - 4\, I(i, j).$$

### 10.2 2D Laplacian Mask Derivation

Add the two 1D second-derivative masks along each axis:

$$\underbrace{\begin{bmatrix}0&0&0\\1&-2&1\\0&0&0\end{bmatrix}}_{\partial^2/\partial x^2} \;+\; \underbrace{\begin{bmatrix}0&1&0\\0&-2&0\\0&1&0\end{bmatrix}}_{\partial^2/\partial y^2} \;=\; \underbrace{\begin{bmatrix}0&1&0\\1&-4&1\\0&1&0\end{bmatrix}}_{\nabla^2}.$$

The 8-connected version (including diagonals) has centre $-8$ and all eight neighbours $+1$.

**Key properties.**

- Coefficients sum to **zero** → flat regions give a zero response.
- **Isotropic** — equal response in all directions (equal weights on all four neighbours).
- **Single mask, cheaper than the gradient.** A gradient detector requires *two* convolutions ($I_x$ and $I_y$) followed by a magnitude computation; the Laplacian needs only **one** convolution.
- **No directional information** — cannot recover edge orientation $\theta$ (a scalar response, not a vector).
- **Very noise-sensitive** — differentiates twice, giving $20 \sigma^2$ amplification in 2D (Section 5.1), making it practically unusable on raw images.

### 10.3 Two Practical Issues with the Bare Laplacian

The two issues below dictate the rest of the Laplacian-based pipeline:

1. **Sensitivity to fine detail and noise.** Twice differentiation amplifies high-frequency content massively. *Fix:* **smooth before differentiating** with a Gaussian → leads to the LoG (§11).
2. **Responds equally to strong and weak edges.** A zero-crossing exists wherever $\nabla^2 I$ changes sign — including across noise-induced ripples that have no perceptual significance. *Fix:* **suppress zero-crossings with low gradient magnitude** (§13.3) so only sign changes coincident with a true intensity transition survive.

---

## 11. Laplacian of Gaussian (LoG)

### 11.1 Key Identity

$$\nabla^2 [f * G_\sigma] = f * \nabla^2 G_\sigma.$$

**Proof.** Write convolution:

$$\nabla^2 [f * G] = \nabla^2 \int f(u, v)\, G(x - u,\, y - v)\, du\, dv.$$

Since $f(u, v)$ does not depend on $(x, y)$, $\nabla^2$ passes inside the integral:

$$= \int f(u, v)\, \nabla^2 G(x - u,\, y - v)\, du\, dv \;=\; f * \nabla^2 G. \qquad \square$$

### 11.2 The LoG Kernel — Closed Form

$$\mathrm{LoG}(x, y) = \frac{-1}{\pi \sigma^4}\left(1 - \frac{x^2 + y^2}{2 \sigma^2}\right) e^{-\tfrac{x^2 + y^2}{2 \sigma^2}}.$$

Three factors:

- **Prefactor** $\dfrac{-1}{\pi \sigma^4}$ — a negative scaling.
- **Polynomial** $\left(1 - \dfrac{r^2}{2 \sigma^2}\right)$ — changes sign at $r = \sigma\sqrt{2}$.
- **Gaussian** $e^{-r^2 / 2 \sigma^2}$ — always positive, ensures spatial decay.

**Mexican hat / sombrero shape.**

- Negative at the centre where $r < \sigma\sqrt{2}$.
- **Zero at $r = \sigma\sqrt{2}$ — this is the edge location.**
- Positive in the surrounding ring where $r > \sigma\sqrt{2}$.

**Mandatory constraint.** All kernel values must sum to **exactly zero**.

---

## 12. LoG Kernels at Three Scales

### 12.1 Kernel 1 — $\sigma = 1.0$, $3 \times 3$

Prefactor: $\dfrac{-1}{\pi}$. Evaluate at $(x, y) \in \{-1, 0, +1\}^2$. The corners have $r^2 = 2 = 2 \sigma^2$, so they fall exactly on the zero-crossing ring:

$$K_{\sigma = 1} = \begin{bmatrix}+4 & -1 & +4\\-1 & -13 & -1\\+4 & -1 & +4\end{bmatrix}.$$

Zero-crossing at $r = 1 \cdot \sqrt{2} \approx 1.41$ px ✓

### 12.2 Kernel 2 — $\sigma = \sqrt{2} \approx 1.4$, $5 \times 5$

Prefactor: $\dfrac{-1}{4 \pi}$. The axis-distance-2 positions have $r^2 = 4 = 2 \sigma^2$ — again exactly on the zero-crossing ring, giving $\mathrm{LoG} = 0$:

$$K_{\sigma = \sqrt{2}} = \begin{bmatrix}0 & 0 & -1 & 0 & 0\\0 & -1 & -2 & -1 & 0\\-1 & -2 & 16 & -2 & -1\\0 & -1 & -2 & -1 & 0\\0 & 0 & -1 & 0 & 0\end{bmatrix}.$$

This is the **standard 5×5 LoG kernel** in the literature. Zero-crossing at $r = \sqrt{2} \cdot \sqrt{2} = 2.0$ px ✓

### 12.3 Kernel 3 — $\sigma = 2.0$, $7 \times 7$

Prefactor: $\dfrac{-1}{16 \pi}$. Zero-crossing at $r = 2\sqrt{2} \approx 2.83$ px:

$$K_{\sigma = 2} \approx \frac{1}{1000}\begin{bmatrix}+5 & +5 & +4 & +3 & +4 & +5 & +5\\+5 & +2 & -1 & -3 & -1 & +2 & +5\\+4 & -1 & -8 & -11 & -8 & -1 & +4\\+3 & -3 & -11 & -15 & -11 & -3 & +3\\+4 & -1 & -8 & -11 & -8 & -1 & +4\\+5 & +2 & -1 & -3 & -1 & +2 & +5\\+5 & +5 & +4 & +3 & +4 & +5 & +5\end{bmatrix}.$$

Sum $\approx 0$ ✓

### 12.4 Summary of LoG Kernel Properties

| | $\sigma = 1.0$, $3\times3$ | $\sigma = \sqrt{2}$, $5\times5$ | $\sigma = 2.0$, $7\times7$ |
|---|---|---|---|
| **Prefactor** | $-1/\pi$ | $-1/4\pi$ | $-1/16\pi$ |
| **Zero-crossing $r$** | 1.41 px | 2.00 px | 2.83 px |
| **Detects edges at** | Fine scale | Standard | Coarse scale |
| **Noise sensitivity** | High | Moderate | Low |

---

## 13. LoG Thresholding — Complete Procedure

### 13.1 Why *not* threshold $|R| > \tau$ directly?

The LoG response oscillates between positive and negative values. Thresholding the response *magnitude* picks up the ridges on **both sides** of the edge, producing **two parallel lines per edge** instead of the edge itself.

### 13.2 Stage 1 — Detect Zero-Crossings

Compute $R = f * \nabla^2 G_\sigma$. Pixel $(i, j)$ is a candidate edge if

$$R(i, j) > 0 \;\text{ and }\; R(i, j+1) < 0 \quad \text{(or any adjacent pair sign change)}.$$

Strength of the zero-crossing:

$$\text{strength} = |R(p) - R(q)|,$$

where $p$ and $q$ straddle the crossing. A large difference means a sharp, confident edge.

### 13.3 Stage 2 — Gradient-Magnitude Gating

$$\text{Keep zero-crossing at } (x, y) \;\Longleftrightarrow\; |\nabla I|(x, y) > \tau.$$

**Intuition.** A true edge is a zero-crossing in $\nabla^2 I$ **and** has large $|\nabla I|$ on either side. A noise fluctuation is a zero-crossing but with tiny $|\nabla I|$.

### 13.4 Full LoG Pipeline

```
I(x,y)
   │
   ▼
R = I * ∇²G_σ              (LoG in a single convolution)
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

---

## 14. Difference of Gaussians (DoG)

$$\mathrm{DoG}(x, y) = G_{\sigma_1}(x, y) - G_{\sigma_2}(x, y), \qquad \sigma_1 < \sigma_2.$$

**Why DoG approximates LoG.** The link is the identity

$$\boxed{\;\frac{\partial G_\sigma}{\partial \sigma} \;=\; \sigma\, \nabla^2 G_\sigma.\;}$$

**Proof.** Let $r^2 = x^2 + y^2$ and write the 2D Gaussian as

$$G_\sigma(x, y) = \frac{1}{2 \pi \sigma^2}\, \exp\!\left(-\frac{r^2}{2 \sigma^2}\right).$$

*Differentiate w.r.t. $\sigma$* (product rule on the prefactor and the exponent):

$$\frac{\partial G_\sigma}{\partial \sigma} = \underbrace{\left(-\frac{2}{2 \pi \sigma^3}\right) e^{-r^2 / 2\sigma^2}}_{\text{from prefactor } 1/(2\pi\sigma^2)} \;+\; \underbrace{\frac{1}{2\pi\sigma^2} \cdot \frac{r^2}{\sigma^3}\, e^{-r^2/2\sigma^2}}_{\text{from exponent}} = \frac{1}{\pi \sigma^3}\!\left(\frac{r^2}{2 \sigma^2} - 1\right) e^{-r^2 / 2\sigma^2}.$$

*Compute the Laplacian.* From $\partial_x G = -\tfrac{x}{\sigma^2}\, G$,

$$\frac{\partial^2 G}{\partial x^2} = \left(\frac{x^2}{\sigma^4} - \frac{1}{\sigma^2}\right) G, \qquad \frac{\partial^2 G}{\partial y^2} = \left(\frac{y^2}{\sigma^4} - \frac{1}{\sigma^2}\right) G,$$

so

$$\nabla^2 G_\sigma = \left(\frac{r^2}{\sigma^4} - \frac{2}{\sigma^2}\right) G_\sigma = \frac{1}{\pi \sigma^4}\!\left(\frac{r^2}{2 \sigma^2} - 1\right) e^{-r^2 / 2\sigma^2}.$$

*Compare.* The two expressions differ only in the prefactor — $\sigma^{-3}$ vs $\sigma^{-4}$ — so

$$\frac{\partial G_\sigma}{\partial \sigma} = \sigma\, \nabla^2 G_\sigma. \qquad \square$$

**Finite-difference consequence.** A first-order Taylor expansion in $\sigma$ then gives

$$G_{\sigma + \Delta \sigma} - G_\sigma \;\approx\; \Delta \sigma \cdot \frac{\partial G_\sigma}{\partial \sigma} \;=\; (\sigma\, \Delta \sigma)\, \nabla^2 G_\sigma,$$

i.e. the difference of two nearby Gaussians is proportional to the LoG of the smaller-scale Gaussian — with proportionality constant $\sigma\, \Delta\sigma$.

DoG is cheaper than LoG (two blurs, no explicit second derivative) and forms the basis of **SIFT** for multi-scale keypoint detection.

---

## 15. Detector Comparison

| Property | Sobel | Canny | LoG (Marr–Hildreth) |
|---|---|---|---|
| **Derivative order** | 1st | 1st | 2nd |
| **Noise handling** | Mild $[1, 2, 1]$ | Explicit $G_\sigma$ | $G_\sigma$ embedded in LoG |
| **Noise amplification** | $2\sigma^2$ | $2\sigma^2$ | $6\sigma^2$ (1D), $20\sigma^2$ (2D) |
| **Edge thickness** | Thick (multi-px) | 1-pixel (NMS) | 1-pixel (zero-crossing) |
| **Thresholding** | Single $\tau$ on $G$ | Dual hysteresis $(T_{\text{lo}}, T_{\text{hi}})$ | Zero-crossing + $\|\nabla I\| > \tau$ |
| **Edge connectivity** | None | Yes (hysteresis) | Moderate |
| **Directional info** | Yes ($\theta$) | Yes ($\theta$) | No |
| **Corner handling** | Good | Good | Poor (rounds corners) |
| **Scale parameter** | None | $\sigma$ | $\sigma$ |
| **Best for** | Fast, clean images | General purpose (optimal) | Non-sharp edges, SIFT features |

### 15.1 Gradient vs LoG — Practical Guidance

The two families are complementary, and the right choice depends on the image and the kind of edge being sought:

- **Gradient detectors** (Sobel, Canny) work well when the image contains **sharp intensity transitions** and **low noise** — magnitude peaks are tall, narrow, and easy to threshold.
- **LoG zero-crossings** offer **better localisation**, particularly when **edges are not sharp** (gradual ramps, defocused boundaries). A zero-crossing is a topological event — its position is unaffected by the slope of the ramp, whereas a gradient peak shifts along a wide ramp depending on the threshold.

In short: use the gradient when contrast is high and edges are crisp; switch to the LoG (or DoG) when edges are diffuse or when sub-pixel localisation matters.

---

## 16. Conceptual Map

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