# Image Filtering and the Fourier Transform

*EE5178 — Modern Computer Vision*

---

## 1. Noise in Images

### 1.1 Sources of Noise

Noise in images originates from four major sources:

- **Capture noise** — Hardware-level issues such as poor lighting, sensor thermal fluctuations, and lens obstructions (dust, fog).
- **Sampling noise** — When a continuous real-world scene is discretized into pixels, high frequencies can "fold" into low ones, producing **aliasing** (Moiré-style interference patterns).
- **Processing noise** — Rounding and approximation errors introduced during mathematical operations within the image-processing pipeline.
- **Compression noise** — Lossy formats such as JPEG deliberately discard high-frequency components, producing visible artifacts (ringing, blockiness).

### 1.2 Types of Noise

Three primary noise models are used in computer vision:

| Noise Type | Also Called | Cause | Distribution |
|---|---|---|---|
| **Salt & Pepper** | Impulse noise | Faulty camera sensors | Random binary (white/black pixels) |
| **Gaussian noise** | — | Lighting, sensor electronics, interference | Normal distribution |
| **Shot noise** | Quantum / Poisson noise | Variation in photon count hitting the sensor | Poisson distribution |

**Gaussian noise** is the most common in practice and is described by

$$\varphi(z) = \frac{1}{\sigma\sqrt{2\pi}}\, e^{-\frac{(z-\mu)^2}{2\sigma^2}}.$$

**Shot noise** is unique because its randomness is tied to the physics of photon arrival — it is inherently **Poisson**, although it converges to Gaussian at high photon counts.

---

## 2. Noise Removal Filters

### 2.1 Gaussian Noise → Linear Filters

Two averaging-based filters are used to suppress Gaussian noise.

**Box filter** — a uniform average over a neighbourhood:

$$\frac{1}{9}\begin{bmatrix}1&1&1\\1&1&1\\1&1&1\end{bmatrix}.$$

Every neighbouring pixel is weighted equally. Simple and fast, but **blurs edges aggressively**.

**Gaussian filter** — a weighted average, with the centre pixel weighted most:

$$G(x,y) = \frac{1}{2\pi\sigma^2}\, e^{-\frac{x^2+y^2}{2\sigma^2}}.$$

The weight falls off smoothly with distance from the centre. This produces a more **natural, isotropic blur** and causes less ringing than a box filter. The parameter $\sigma$ controls the spread — larger $\sigma$ means more blur.

### 2.2 Salt & Pepper Noise → Median Filter

Gaussian and box filters **fail** on salt-and-pepper noise: averaging mixes the outlier (a black or white spike) with valid neighbours, smearing the corruption rather than removing it.

The **median filter** replaces each pixel with the **median** of its neighbourhood:

- Sort all neighbouring pixel values → pick the middle one.
- Outlier spikes (0 or 255) sort to the extreme ends and are never chosen.
- **Edges are well preserved**, because the median is always a value that actually exists in the neighbourhood — not an artificial average.

> **Key intuition.** The mean is sensitive to outliers; the median is **robust** to them. This is why the median filter is the standard remedy for impulse noise.

---

## 3. The Fourier Transform

The **2D Fourier Transform** decomposes an image into a weighted sum of **sinusoidal plane waves**. It moves an image from the spatial domain to the frequency domain:

- **Spatial domain** — *where* intensities are (pixel values at locations $(x, y)$).
- **Frequency domain** — *how fast* intensities change (frequency components $F(u, v)$).

### 3.1 1D Discrete Fourier Transform

The 1D **Discrete Fourier Transform (DFT)** decomposes a signal of $N$ samples into $N$ complex frequency components:

$$X_k = \sum_{n=0}^{N-1} x_n \cdot e^{-i 2\pi k n / N}.$$

The **inverse DFT** perfectly reconstructs the original signal:

$$x_n = \frac{1}{N}\sum_{k=0}^{N-1} X_k \cdot e^{i 2\pi k n / N}.$$

Each $X_k$ is a **complex number** — its magnitude tells how much of frequency $k$ is present, and its phase tells *where* in the signal that oscillation is positioned.

### 3.2 2D Continuous Fourier Transform

$$F(u,v) = \iint_{-\infty}^{\infty} f(x,y)\, e^{-j 2\pi (ux + vy)}\, dx\, dy.$$

### 3.3 2D Discrete Fourier Transform

An image $f(x, y)$ of size $M \times N$ is transformed via

$$F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} f(x,y)\, e^{-j 2\pi \left(\tfrac{u x}{M} + \tfrac{v y}{N}\right)},$$

and inverted via

$$f(x,y) = \frac{1}{MN}\sum_{u=0}^{M-1}\sum_{v=0}^{N-1} F(u,v)\, e^{\,j 2\pi \left(\tfrac{u x}{M} + \tfrac{v y}{N}\right)}.$$

Each basis function $e^{-j 2\pi (ux/M + vy/N)}$ is a **2D sinusoidal plane wave** whose spatial frequency is $(u/M,\, v/N)$ and whose orientation is set by the direction of the vector $(u, v)$.

> **Practical note on discretization.** Real images are discrete 8-bit arrays (values 0–255). The continuous integral is replaced by a double sum. Sampling is modelled by a **2D impulse train** — a grid of Dirac delta functions that "picks out" pixel values.

### 3.4 Why Use the Frequency Domain?

| Application | Reason |
|---|---|
| **Anti-aliasing** | Remove high frequencies *before* downsampling to prevent Moiré patterns. |
| **Compression** | JPEG discards high-frequency content imperceptible to the human eye. |
| **Efficiency** | The convolution theorem turns spatial convolution into pointwise multiplication. |
| **Denoising** | Precisely remove periodic patterns or sensor-grid noise. |

---

## 4. Properties of the Fourier Transform

### 4.1 Linearity

$$\mathcal{F}\{a f(x,y) + b g(x,y)\} = a F(u,v) + b G(u,v).$$

The transform of a weighted sum of images is the weighted sum of their individual transforms.

### 4.2 Separability

$$F(u,v) = \sum_{x=0}^{M-1}\left[\sum_{y=0}^{N-1} f(x,y)\, e^{-j 2\pi \tfrac{v y}{N}}\right] e^{-j 2\pi \tfrac{u x}{M}}.$$

The 2D transform can be computed by

1. performing 1D DFTs along all **rows** (inner sum), and then
2. performing 1D DFTs along all **columns** of the result (outer sum).

This is the basis for massive computational savings in the FFT.

### 4.3 Translation (Spatial Shift)

$$f(x - x_0,\, y - y_0) \;\longleftrightarrow\; F(u,v)\, e^{-j 2\pi \left(\tfrac{u x_0}{M} + \tfrac{v y_0}{N}\right)}.$$

A spatial shift multiplies $F(u,v)$ by a complex exponential of unit magnitude — the **phase changes** but $|F(u,v)|$ stays identical. This is why the magnitude spectrum is used for pattern matching tasks that must be position-independent.

### 4.4 Scaling

$$f(a x,\, b y) \;\longleftrightarrow\; \frac{1}{|ab|}\, F\!\left(\frac{u}{a},\, \frac{v}{b}\right).$$

Stretching an image spatially causes **reciprocal shrinking** in the frequency domain (and vice versa). Formally, spatial width × frequency bandwidth = constant.

### 4.5 Conjugate Symmetry

$$F(u,v) = F^*(-u,-v).$$

For any real-valued image (all standard digital images), the Fourier transform is **Hermitian symmetric** — the left half of the spectrum is the complex-conjugate mirror of the right half. Only half the spectrum carries unique information, which many FFT libraries exploit automatically.

### 4.6 Rotation

$$f(r,\, \phi + \theta) \;\longleftrightarrow\; F(\rho,\, \psi + \theta).$$

Rotating the image by angle $\theta$ in the spatial domain **rotates the spectrum by the same angle $\theta$** in the frequency domain. Oriented structures in the image therefore produce oriented streaks in the spectrum.

### 4.7 Periodicity

**Frequency domain:** $F(u, v) = F(u + k_1 M,\; v + k_2 N).$

**Spatial domain:** $f(x, y) = f(x + k M,\; y + l N).$

The DFT and its inverse are **infinitely periodic**: the transform repeats every $M$ units horizontally and $N$ units vertically, because the DFT implicitly tiles the image as one cell of an infinite 2D mosaic.

### 4.8 Differentiation

$$\frac{\partial f(x,y)}{\partial x} \;\longleftrightarrow\; j 2\pi u \cdot F(u,v), \qquad \frac{\partial f(x,y)}{\partial y} \;\longleftrightarrow\; j 2\pi v \cdot F(u,v).$$

Differentiating in space is equivalent to **multiplying by a frequency ramp** — the higher the frequency, the more it is amplified. This makes derivatives high-pass operations.

### 4.9 Summary Table

| Property | Spatial Domain | Frequency Domain | Intuition |
|---|---|---|---|
| Linearity | $a f + b g$ | $a F + b G$ | Superposition holds. |
| Separability | 2D operation | 1D DFT on rows, then columns | Computational saving. |
| Translation | $f(x - x_0,\, y - y_0)$ | $F(u,v)\, e^{-j 2\pi (u x_0 / M + v y_0 / N)}$ | Phase shifts; magnitude unchanged. |
| Scaling | $f(a x,\, b y)$ | $\tfrac{1}{|ab|}\, F(u/a,\, v/b)$ | Stretch ↔ compression. |
| Conjugate symmetry | Real image | $F(u,v) = F^*(-u,-v)$ | Half spectrum is unique. |
| Rotation | Rotate by $\theta$ | Spectrum rotates by $\theta$ | Oriented features → oriented spectral streaks. |
| Convolution | $f * g$ | $F \cdot G$ | Convolution = pointwise multiply. |
| Differentiation | $\partial f / \partial x$ | $j 2\pi u \cdot F$ | Derivatives = ramp multiplication. |
| Periodicity | Finite image tile | Infinitely periodic spectrum | DFT tiles the image. |

---

## 5. Spectral Artifacts and Windowing

### 5.1 The Spectral Cross

Because the DFT tiles the image periodically, a sharp jump occurs at image boundaries when the left edge does not match the right edge (or top does not match bottom). In the frequency domain, this sharp jump appears as a broadband high-frequency pulse — a bright **horizontal-and-vertical cross** through the centre of the magnitude spectrum.

### 5.2 Windowing Fix

Multiply the image by a **window function** $w(x, y)$ that tapers smoothly to zero at all edges before computing the DFT:

$$f_w(x, y) = w(x, y)\, f(x, y).$$

This eliminates the boundary jump, suppresses the cross artifact, and reveals the true frequency content of the image.

---

## 6. Convolution Theorem and Fast Filtering

### 6.1 The Convolution Theorem

$$\mathcal{F}\{f(x,y) * g(x,y)\} = F(u,v) \cdot G(u,v).$$

Spatial convolution — applying a filter kernel to every pixel — is equivalent to **pointwise multiplication** in the frequency domain. The FFT-based filtering pipeline is therefore:

1. **FFT** of image $\to F(u, v)$.
2. **FFT** of filter kernel $\to H(u, v)$.
3. **Pointwise multiply** $\to G(u, v) = F(u, v) \cdot H(u, v)$.
4. **Inverse FFT** $\to$ filtered image $g(x, y)$.

For large kernels this is **vastly faster** than direct spatial convolution.

### 6.2 Circular vs. Linear Convolution (Wrap-Around)

Because the DFT assumes periodicity, the Convolution Theorem actually gives **circular convolution** — pixels that exit one edge wrap around and bleed into the opposite edge, producing a *ghosting* artifact:

$$(f \circledast g)(x, y) = \sum_{m, n} f(m, n)\, g\!\left((x - m)_M,\; (y - n)_N\right),$$

where $(x - m)_M$ denotes modulo-$M$ wrap-around.

**Fix.** **Zero-pad** the image (append a border of zeros) before computing the FFT. The padding provides a buffer zone so the periodic copies of the image do not interact with the filter response, yielding standard **linear convolution**.

---

## 7. Computational Complexity — DFT vs. FFT

### 7.1 Why the Naive DFT Is Expensive

For an $N \times N$ image, the brute-force DFT computes $N^2$ frequency points, each requiring a sum over all $N^2$ pixels:

$$\underbrace{N^2}_{\text{points}} \times \underbrace{N^2}_{\text{summations per point}} = O(N^4).$$

### 7.2 The FFT Algorithm

The **Fast Fourier Transform** exploits two key insights:

1. **Separability.** The 2D DFT decomposes into 1D DFTs along rows, then columns.
2. **Divide-and-conquer.** Each 1D DFT of length $N$ is recursively split into even- and odd-indexed sub-problems:

$$X_k = \underbrace{E_k}_{\text{even-indexed terms}} + \underbrace{e^{-j 2\pi k / N}}_{\text{twiddle factor}} \cdot \underbrace{O_k}_{\text{odd-indexed terms}}.$$

The **twiddle factor** $e^{-j 2\pi k / N}$ is periodic and symmetric, so computed values are reused across multiple $k$. This reduces 1D DFT cost from $O(N^2)$ to $O(N \log N)$.

### 7.3 Total 2D FFT Cost

$$\underbrace{2N}_{\text{row + column passes}} \times \underbrace{O(N \log N)}_{\text{per 1D FFT}} = O(N^2 \log N).$$

For a $1024 \times 1024$ image, this is a dramatic improvement: $O(N^4) \approx 10^{12}$ operations versus $O(N^2 \log N) \approx 10^7$ operations.

---

## 8. Reading the Frequency Spectrum

### 8.1 Sinusoidal Patterns

| Spatial image | Frequency spectrum |
|---|---|
| Vertical stripes (high-frequency along $x$) | Two bright dots on the horizontal axis |
| Horizontal stripes (high-frequency along $y$) | Two bright dots on the vertical axis |
| Diagonal stripes | Two dots at the corresponding $45^\circ$ angle |
| Checkerboard pattern | Four dots at the corners |

This follows from the Fourier transform of a pure sinusoid:

$$\mathcal{F}\{\sin(2\pi A t)\} = \frac{1}{2 i}\left[\delta(f - A) - \delta(f + A)\right],$$

giving spectral peaks exactly at $\pm A$.

### 8.2 Real Images

- **Clean portrait** — diffuse cloud centred at DC; natural images carry most of their energy in low frequencies, with a faint spectral cross from boundary discontinuity.
- **Clean cameraman** — diagonal rays in the spectrum corresponding to tripod legs and other oriented edges.
- **Cameraman + Gaussian noise** — elevated high-frequency floor (bright outer regions).
- **Cameraman + heavy noise** — spectrum becomes nearly uniform; noise is spectrally flat.
- **Heavily degraded image** — almost all structure lost from the spectrum; only DC remains strong.

### 8.3 What the Spectrum Tells You at a Glance

| Feature in spectrum | Meaning in image |
|---|---|
| Bright centre (DC component) | High mean brightness; dominant low-frequency structure. |
| Diffuse cloud near centre | Smooth regions, natural image with mostly coarse features. |
| Oriented streaks at angle $\theta$ | Strong edges or structures oriented at angle $\theta$ in the image. |
| Elevated outer regions | High-frequency noise or fine texture. |
| Nearly uniform spectrum | Heavily corrupted image; noise overwhelms structure. |
| Two bright dots on horizontal axis | Vertical stripes (sinusoid in $x$-direction). |
| Two bright dots on vertical axis | Horizontal stripes (sinusoid in $y$-direction). |

### 8.4 Low-pass vs. High-pass Filters in Frequency Domain

- **Gaussian blur (low-pass).** Its spectrum is a smooth blob centred at DC — it passes low frequencies and attenuates high ones, which is why it blurs edges and removes noise.
- **Edge detection (high-pass).** Its spectrum is zero at DC and grows outward — it suppresses smooth regions and amplifies abrupt changes (edges, texture).

### 8.5 Filtering Pipeline in the Frequency Domain

1. Compute the FFT of the image: $F(u, v) = \mathcal{F}\{f(x, y)\}$.
2. Compute the FFT of the filter: $H(u, v) = \mathcal{F}\{h(x, y)\}$.
3. Pointwise multiply: $G(u, v) = F(u, v) \cdot H(u, v)$.
4. Apply the inverse FFT: $g(x, y) = \mathcal{F}^{-1}\{G(u, v)\}$.

**Example — Gaussian blur.** In the frequency domain, multiply the image spectrum with a Gaussian blob (low-pass). The result suppresses high-frequency details, producing a smooth, blurred image.