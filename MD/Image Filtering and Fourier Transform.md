# Image Filtering and Fourier Transform
*EE5178 — Modern Computer Vision*

---

## Slide 1 — Title
**Image Filtering and Fourier Transform**

---

## Slide 2 — Types of Noise in Images

**Sources of Noise:**
- **Capture Noise:**
  - *Lighting* — high-lit and low-lit conditions
  - *Sensor* — temperature fluctuations, electrical interference
  - *Lens* — dust, fog, etc.
- **Sampling Noise:** Continuous real-world scene → discrete pixels; can cause aliasing (Moiré patterns)
- **Processing Noise:** Introduced by mathematical approximations during image processing
- **Image Compression Noise:** JPEG lossy compression removes finer high-frequency details

---

## Slide 3 — Salt and Pepper Noise
- Also known as **Impulse Noise**
- Random **white and black pixels** scattered across the image
- Source: faulty camera sensors

*Visual: Grayscale still-life photograph with scattered random bright/dark pixels throughout.*

---

## Slide 4 — Gaussian Noise
- Most **common** type of noise
- Caused by lighting effects, sensor limitations, and electronic interference
- Described by the Gaussian (normal) distribution:

$$\varphi(z) = \frac{1}{\sigma\sqrt{2\pi}}\, e^{-\frac{(z-\mu)^2}{2\sigma^2}}$$

---

## Slide 5 — Shot Noise (Poisson Noise)
- Also known as **Quantum noise**
- Occurs due to variations in the number of photons incident on the sensor
- Randomness follows a **Poisson distribution** (not Gaussian)

---

## Slide 6 — Gaussian Noise Removal
Two filters for removing Gaussian noise:
- **Box Filter**
- **Gaussian Filter**

---

## Slide 7 — Box Filter
Uniform average over a neighborhood:

$$\frac{1}{9}\begin{bmatrix}1&1&1\\1&1&1\\1&1&1\end{bmatrix}$$

Replaces each pixel with the mean of its neighborhood. Simple but blurs edges.

---

## Slide 8 — Gaussian Filter
Weighted average with Gaussian weights — centre pixel weighted more than outer pixels:

$$G(x,y) = \frac{1}{2\pi\sigma^2}\, e^{-\frac{x^2+y^2}{2\sigma^2}}$$

Better than box filter: less ringing, more natural blur.

---

## Slide 9 — Salt and Pepper Noise Removal
*Visual: Cameraman image with salt-and-pepper noise vs. Gaussian-filtered result.*
- Gaussian filter reduces impulse noise but blurs fine details.
- Better choice: **Median filter**

---

## Slide 10 — Median Filtering
- Replaces each pixel with the **median** of its neighborhood (not the mean)
- Very effective for salt-and-pepper noise — outliers are discarded
- Preserves edges better than Gaussian/box filter

*Visual: Salt-and-pepper noisy cameraman → median-filtered result (clean, sharp edges preserved).*

---

## Part 2: Fourier Transform

---

## Slide 11 — Fourier Transform in 1D

**1D Discrete Fourier Transform (DFT):**

$$X_k = \sum_{n=0}^{N-1} x_n \cdot e^{-i2\pi kn/N}$$

**Inverse DFT:**

$$x_n = \frac{1}{N}\sum_{k=0}^{N-1} X_k \cdot e^{i2\pi kn/N}$$

Transforms a signal from the **time/spatial domain** to the **frequency domain**. Reveals which frequencies dominate.

---

## Slide 12 — The Frequency Perspective in 2D Vision
The **2D Fourier Transform** decomposes an image into a weighted sum of **sinusoidal plane waves**.

- **Spatial domain:** *where* intensities are (pixel values at locations (x, y))
- **Frequency domain:** *how fast* intensities change (frequency components F(u, v))

**2D Continuous Fourier Transform:**

$$F(u,v) = \iint_{-\infty}^{\infty} f(x,y)\, e^{-j2\pi(ux+vy)}\, dx\, dy$$

---

## Slide 13 — Discrete Fourier Transform (2D)
In practice, images are discrete (pixel arrays). Discretization uses a **2D Impulse Train** (sampling grid of Dirac delta functions).

- 8-bit depth: light intensity mapped to integers 0–255
- The continuous integral becomes a discrete double sum

**2D DFT pair:**

$$\hat{h}(k,l) = \sum_{n=0}^{N-1}\sum_{m=0}^{M-1} e^{-i(\omega_k n + \omega_l m)}\, h(n,m)$$

$$h(n,m) = \frac{1}{NM}\sum_{k=0}^{N-1}\sum_{l=0}^{M-1} e^{i(\omega_k n + \omega_l m)}\, \hat{h}(k,l)$$

---

## Slide 14 — Why Use the Frequency Domain in 2D?
A core pillar of classical computer vision:

| Application | Reason |
|---|---|
| **Anti-aliasing** | Remove high frequencies *before* downsampling to prevent Moiré patterns |
| **Compression** | JPEG discards high-frequency "noise" imperceptible to the human eye |
| **Efficiency** | Convolution theorem → expensive spatial convolution = fast pointwise multiply in freq. domain |
| **Denoising** | Precisely remove periodic patterns or sensor grid noise |

---

## Slide 15 — 2D Fourier Basis
The 2D DFT represents an image as a sum of **complex exponentials**:

$$F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} f(x,y)\,e^{-j2\pi\!\left(\frac{ux}{M}+\frac{vy}{N}\right)}$$

Each basis function $e^{-j2\pi(ux/M + vy/N)}$ is a complex sinusoidal plane wave with spatial frequency $(u/M,\, v/N)$ and orientation determined by $(u, v)$.

---

## Slides 16–22 — Properties of the Fourier Transform

### Linearity
$$\mathcal{F}\{af(x,y) + bg(x,y)\} = aF(u,v) + bG(u,v)$$
The transform of a weighted sum of images is the weighted sum of their individual transforms.

### Separability
$$F(u,v) = \sum_{x=0}^{M-1}\left[\sum_{y=0}^{N-1} f(x,y)\,e^{-j2\pi\frac{vy}{N}}\right]e^{-j2\pi\frac{ux}{M}}$$
The 2D transform can be computed by:
1. Performing 1D DFTs along all **rows** (inner sum)
2. Then 1D DFTs along all **columns** of the result (outer sum)

### Translation (Spatial Shift)
$$f(x-x_0,\,y-y_0) \iff F(u,v)\,e^{-j2\pi\!\left(\frac{ux_0}{M}+\frac{vy_0}{N}\right)}$$
Shifting an image in the spatial domain only changes the **phase** in the frequency domain — the **magnitude spectrum is unchanged**.

### Scaling
$$f(ax,\,by) \iff \frac{1}{|ab|}F\!\left(\frac{u}{a},\frac{v}{b}\right)$$
Stretching an image in the spatial domain causes **reciprocal shrinking** in the frequency domain (and vice versa).

### Conjugate Symmetry
$$F(u,v) = F^*(-u,-v)$$
For real-valued images f(x, y) (all standard digital images), the Fourier Transform is **Hermitian symmetric** — only half the spectrum carries unique information.

### Rotation
$$f(r,\,\phi+\theta) \iff F(\rho,\,\psi+\theta)$$
Rotating an image by angle θ in the spatial domain **rotates the spectrum by the same angle θ** in the frequency domain.

### Periodicity
**Frequency domain:**
$$F(u,v) = F(u + k_1 M,\;v + k_2 N)$$
**Spatial domain:**
$$f(x,y) = f(x + kM,\;y + lN)$$
The DFT (and its inverse) are **infinitely periodic** — the transform repeats every M units horizontally and N units vertically. This is because the DFT assumes the image is one tile in an infinitely repeating 2D mosaic.

---

## Slide 23 — Edge Discontinuity and Windowing
Since the DFT tiles the image periodically, a sharp jump occurs at image boundaries when the left edge doesn't match the right edge.

**Effect:** A bright **"spectral cross"** (horizontal and vertical lines) appears through the centre of the frequency plot.

**Fix:** Multiply by a **Window Function** w(x, y) that smoothly tapers pixel values to zero at image edges, making the tiled transition seamless.

---

## Slide 24 — Convolution Theorem
$$\mathcal{F}\{f(x,y) * g(x,y)\} = F(u,v) \cdot G(u,v)$$

**Spatial convolution** ↔ **element-wise multiplication in the frequency domain**.

This is the basis for fast filtering in computer vision — instead of O(N²) spatial convolution, compute FFT, multiply, and IFFT.

---

## Slide 25 — Wrap-Around (Ghosting) and Zero-Padding
Because the DFT assumes the image is periodic, the Convolution Theorem performs **Circular Convolution**:

$$(f \circledast g)(x,y) = \sum_{m,n} f(m,n)\,g\!\left((x-m)_M,\,(y-n)_N\right)$$

where $(x-m)_M$ denotes modulo-M wrap-around. If you apply a filter without enough space, pixels that "exit" the right side wrap around and bleed into the left side.

To achieve **Linear Convolution** (standard filtering), zero-pad the image with a border of zeros — this provides a buffer zone preventing interaction with periodic copies.

---

## Slide 26 — Inverse 2D DFT

$$f(x,y) = \frac{1}{MN}\sum_{u=0}^{M-1}\sum_{v=0}^{N-1} F(u,v)\,e^{j2\pi\!\left(\frac{ux}{M}+\frac{vy}{N}\right)}$$

---

## Slide 27 — Differentiation Property

$$\frac{\partial f(x,y)}{\partial x} \iff j2\pi u \cdot F(u,v) \quad \text{and} \quad \frac{\partial f(x,y)}{\partial y} \iff j2\pi v \cdot F(u,v)$$

Differentiating in the spatial domain = multiplying by frequency variable in frequency domain. **Spatial derivative** (rate of change of pixel intensity) corresponds to **ramp multiplication** in frequency domain.

---

## Slide 28 — Computational Cost of DFT

$$N^2 \text{ (points)} \times N^2 \text{ (summations per point)} = O(N^4)$$

The standard DFT is a "brute force" approach. To find the value of just one frequency point (u,v), the algorithm must sum all N×N pixels. With N² frequency points, total cost is **O(N⁴)**.

---

## Slide 29 — Fast Fourier Transform (FFT)

$$\text{DFT: } O(N^4) \quad \longrightarrow \quad \text{FFT: } O(N^2 \log N)$$

The 2D Fast Fourier Transform exploits separability and symmetry to reduce complexity dramatically.

---

## Slide 30 — 2D FFT via Row-Column Decomposition

$$F(u,v) = \sum_{y=0}^{N-1}\left[\sum_{x=0}^{M-1} f(x,y)\,e^{-j2\pi\frac{ux}{M}}\right]e^{-j2\pi\frac{vy}{N}}$$

**1D FFT of Rows** → **1D FFT of Columns**

---

## Slide 31 — 1D FFT Algorithm

$$X_k = \underbrace{E_k}_{\text{Even}} + \underbrace{e^{-j2\pi k/N}}_{\text{Twiddle}} \cdot \underbrace{O_k}_{\text{Odd}}$$

**Total Complexity:** $\underbrace{2N}_{\text{Rows+Cols}} \times \underbrace{O(N\log N)}_{\text{1D FFT}} = O(N^2\log N)$

The divide-and-conquer approach splits data into even/odd samples, exploiting twiddle factor periodicity to reuse computations.

---

## Slides 32–33 — Fourier Transform for Images: DFT Pair

**2D Discrete Fourier Transform:**

$$\hat{h}(k,l) = \sum_{n=0}^{N-1}\sum_{m=0}^{M-1} e^{-i(\omega_k n + \omega_l m)}\, h(n,m)$$

**Inverse 2D DFT:**

$$h(n,m) = \frac{1}{NM}\sum_{k=0}^{N-1}\sum_{l=0}^{M-1} e^{i(\omega_k n + \omega_l m)}\, \hat{h}(k,l)$$

---

## Slides 33–49 — Fourier Transform of Images: Visual Examples

### Sinusoidal Patterns
| Spatial Image | Frequency Spectrum |
|---|---|
| Vertical stripes (high-freq, x-direction) | Two dots on horizontal axis |
| Horizontal stripes (high-freq, y-direction) | Two dots on vertical axis |
| Diagonal stripes | Two dots at 45° |
| Checkerboard pattern | 4 dots at corners |

**Key insight:** A pure sinusoid with frequency A gives spectral peaks at ±A:
$$\mathfrak{F}\{\sin(2\pi At)\} = \frac{1}{2i}[\delta(f-A) - \delta(f+A)]$$

### Real Images
- **Portrait (clean):** Spectrum shows a diffuse cloud centred at DC (0,0) — natural images have most energy in low frequencies; faint spectral cross from boundary discontinuity
- **Cameraman (clean):** Spectrum shows diagonal rays corresponding to the tripod legs and other oriented edges
- **Cameraman + Gaussian noise:** Spectrum shows elevated high-frequency noise floor (bright outer regions)
- **Cameraman + heavy noise:** Spectrum nearly uniform (noise is spectrally flat)
- **Heavily degraded image:** Almost all structure lost from spectrum; only DC remains strong

### Observation
- **Low frequencies** (centre of spectrum) → smooth regions, coarse structure
- **High frequencies** (outer regions) → fine detail, edges, noise
- Oriented structures in the image → oriented streaks in the spectrum

---

## Slides 50–51 — Fourier Transform of Filters

Filter kernels also have a frequency domain representation:
- **Low-pass filter (Gaussian):** Spectrum is a smooth blob centred at DC — passes low frequencies, suppresses high
- **High-pass filter:** Spectrum has zero at DC and grows toward high frequencies

---

## Slides 52–53 — Filtering Operation in Frequency Domain

**Pipeline:**
1. Compute FFT of image: $F(u,v) = \mathcal{F}\{f(x,y)\}$
2. Compute FFT of filter: $H(u,v) = \mathcal{F}\{h(x,y)\}$
3. Pointwise multiply: $G(u,v) = F(u,v) \cdot H(u,v)$
4. Apply Inverse FFT: $g(x,y) = \mathcal{F}^{-1}\{G(u,v)\}$

*Visual: Low-pass Gaussian filter in frequency domain (bright circular blob) multiplied with image spectrum → blurred output (recovered by IFT).*

**Example (Gaussian blur):**
- Frequency domain: multiply image spectrum with a Gaussian blob (low-pass)
- Result: suppress high-frequency details → smooth/blurred image

---

## Slide 54 — Summary of Properties

| Property | Spatial Domain | Frequency Domain |
|---|---|---|
| Linearity | $af + bg$ | $aF + bG$ |
| Separability | Row + col convolution | Row + col 1D DFT |
| Translation | $f(x-x_0, y-y_0)$ | Phase shift, same magnitude |
| Scaling | Stretch | Reciprocal compression |
| Conjugate symmetry | Real image | $F(-u,-v) = F^*(u,v)$ |
| Rotation | Rotate by θ | Spectrum rotates by θ |
| Convolution | $f * g$ | $F \cdot G$ |
| Differentiation | $\partial f / \partial x$ | $j2\pi u \cdot F$ |
| Periodicity | Finite image tile | Infinite periodic spectrum |

Here are comprehensive notes for the full lecture on **Image Filtering and the Fourier Transform** (EE5178 — Modern Computer Vision):

***

# Part 1: Noise in Images

## Sources of Noise

Noise in images originates from four major sources: [geeksforgeeks](https://www.geeksforgeeks.org/computer-vision/fast-fourier-transform-in-image-processing/)

- **Capture noise**: Hardware-level issues — poor lighting, sensor thermal fluctuations, and lens obstructions (dust, fog)
- **Sampling noise**: When the real-world continuous scene is discretized into pixels; this can cause **aliasing** — Moiré-style interference patterns where high frequencies "fold" into lower ones
- **Processing noise**: Rounding/approximation errors introduced during mathematical operations within the image processing pipeline
- **Compression noise**: Lossy formats like JPEG deliberately discard high-frequency components, introducing visible artifacts (ringing, blockiness)

## Types of Noise

There are three primary noise models used in computer vision:

| Noise Type | Also Called | Cause | Distribution |
|---|---|---|---|
| **Salt & Pepper** | Impulse noise | Faulty camera sensors | Random binary (white/black pixels) |
| **Gaussian noise** | — | Lighting, sensor electronics, interference | Normal distribution \(\varphi(z) = \frac{1}{\sigma\sqrt{2\pi}} e^{-\frac{(z-\mu)^2}{2\sigma^2}}\) |
| **Shot noise** | Quantum/Poisson noise | Variation in photon count hitting sensor | Poisson distribution |

Gaussian noise is the most **common** in practice. Shot noise is unique because its randomness is tied to the physics of photon arrival — it is inherently **Poisson**, not Gaussian, though they converge at high photon counts. [geeksforgeeks](https://www.geeksforgeeks.org/computer-vision/fast-fourier-transform-in-image-processing/)

***

# Part 2: Noise Removal Filters

## Gaussian Noise → Linear Filters

Two filters are used to suppress Gaussian noise, both based on **averaging**: [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

**Box Filter** — Uniform average over a neighborhood:

\[
\frac{1}{9}\begin{bmatrix}1&1&1\\1&1&1\\1&1&1\end{bmatrix}
\]

Every neighboring pixel is weighted equally. It is simple and fast but **blurs edges** aggressively.

**Gaussian Filter** — Weighted average, with the center pixel weighted most:

\[
G(x,y) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2+y^2}{2\sigma^2}}
\]

The weight falls off smoothly with distance from center. This produces a more **natural, isotropic blur** and causes less ringing than a box filter. The parameter \(\sigma\) controls the spread — larger \(\sigma\) = more blur. [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

## Salt & Pepper Noise → Median Filter

Gaussian and box filters **fail** at salt-and-pepper noise because they average in the outlier (the white/black spike) along with valid neighbors, spreading the corruption rather than removing it. [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

The **Median Filter** replaces each pixel with the **median** of its neighborhood:

- Sort all neighboring pixel values → pick the middle value
- Outlier spikes (0 or 255) sort to the extreme ends and are never chosen as median
- **Edges are well preserved** because the median tends to be a value that actually exists in the neighborhood, not an artificial average

> Key intuition: The mean is sensitive to outliers; the median is **robust** to them. This is why the median filter is the standard remedy for impulse noise.

***

# Part 3: Fourier Transform

## 1D DFT — The Foundation

The 1D **Discrete Fourier Transform** decomposes a signal of \(N\) samples into \(N\) complex frequency components: [geeksforgeeks](https://www.geeksforgeeks.org/computer-vision/fast-fourier-transform-in-image-processing/)

\[
X_k = \sum_{n=0}^{N-1} x_n \cdot e^{-i2\pi kn/N}
\]

The **Inverse DFT** perfectly reconstructs the original signal:

\[
x_n = \frac{1}{N}\sum_{k=0}^{N-1} X_k \cdot e^{i2\pi kn/N}
\]

Each \(X_k\) is a **complex number** — its magnitude tells you how much of frequency \(k\) is present, and its phase tells you where in the signal that oscillation is positioned.

## 2D DFT — Images

An image \(f(x,y)\) of size \(M \times N\) is transformed via: [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

\[
F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} f(x,y)\, e^{-j2\pi\!\left(\frac{ux}{M}+\frac{vy}{N}\right)}
\]

And inverted via:

\[
f(x,y) = \frac{1}{MN}\sum_{u=0}^{M-1}\sum_{v=0}^{N-1} F(u,v)\, e^{j2\pi\!\left(\frac{ux}{M}+\frac{vy}{N}\right)}
\]

Each basis function \(e^{-j2\pi(ux/M + vy/N)}\) is a **2D sinusoidal plane wave** whose spatial frequency is \((u/M, v/N)\) and whose orientation is set by the direction of the vector \((u, v)\).

**Practical note on discretization**: Real images are discrete 8-bit arrays (values 0–255). The continuous integral is replaced by a double sum. Sampling is modeled by a **2D impulse train** (a grid of Dirac delta functions) that "picks out" pixel values.

***

# Part 4: Properties of the Fourier Transform

These are the core mathematical properties that make the FT powerful in vision: [homepages.inf.ed.ac](https://homepages.inf.ed.ac.uk/rbf/HIPR2/fourier.htm)

| Property | Spatial Domain | Frequency Domain | Intuition |
|---|---|---|---|
| **Linearity** | \(af + bg\) | \(aF + bG\) | FT is a linear operator; superposition holds |
| **Separability** | 2D image | 1D DFT on rows, then columns | Massive computational saving |
| **Translation** | \(f(x-x_0, y-y_0)\) | \(F(u,v)\cdot e^{-j2\pi(ux_0/M + vy_0/N)}\) | Shift = phase change only; magnitude unchanged |
| **Scaling** | \(f(ax, by)\) | \(\frac{1}{|ab|}F(u/a, v/b)\) | Stretch in space ↔ compression in frequency |
| **Conjugate Symmetry** | Real image | \(F(u,v) = F^*(-u,-v)\) | Only half the spectrum is unique |
| **Rotation** | Rotate by \(\theta\) | Spectrum rotates by \(\theta\) | Oriented structures → oriented spectral streaks |
| **Convolution** | \(f * g\) | \(F \cdot G\) | Convolution = pointwise multiply in freq. domain |
| **Differentiation** | \(\partial f / \partial x\) | \(j2\pi u \cdot F(u,v)\) | Derivatives = ramp multiplication |
| **Periodicity** | Finite image tile | Infinitely periodic spectrum | DFT implicitly tiles the image |

### Translation: Why Magnitude Is Shift-Invariant
A spatial shift multiplies \(F(u,v)\) by a complex exponential of unit magnitude — the **phase changes** but \(|F(u,v)|\) stays identical  [homepages.inf.ed.ac](https://homepages.inf.ed.ac.uk/rbf/HIPR2/fourier.htm). This is why the magnitude spectrum is used for pattern matching tasks that must be position-independent.

### Scaling: The Bandwidth Trade-off
If you **zoom into** (stretch) an image spatially, its frequency content gets **compressed** toward DC (lower frequencies dominate). Conversely, a small, detailed patch has its energy spread across higher frequencies. Formally: spatial width × frequency bandwidth = constant. [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

### Conjugate Symmetry: Halving the Work
For any real-valued image, \(F(-u,-v) = F^*(u,v)\). This means the spectrum has **Hermitian symmetry** — the left half is the complex conjugate mirror of the right half. You only need to store/compute half the spectrum, which many FFT libraries exploit automatically.

***

# Part 5: Spectral Artifacts and Windowing

## The Spectral Cross

The DFT assumes the image tiles infinitely. When the left and right edges don't match in intensity, tiling creates a **sharp discontinuity**. In the frequency domain, this sharp jump looks like a broadband high-frequency pulse, which appears as a **bright horizontal and vertical cross** through the center of the magnitude spectrum. [micro.magnet.fsu](https://micro.magnet.fsu.edu/primer/java/digitalimaging/processing/fouriertransform/index.html)

## Windowing Fix

Multiply the image by a **window function** \(w(x,y)\) that tapers smoothly to zero at all edges before computing the DFT. This eliminates the jump, suppresses the cross artifact, and reveals the true frequency content. [micro.magnet.fsu](https://micro.magnet.fsu.edu/primer/java/digitalimaging/processing/fouriertransform/index.html)

***

# Part 6: Convolution Theorem and Fast Filtering

## The Convolution Theorem

This is arguably the most practically important result in the entire lecture: [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

\[
\mathcal{F}\{f(x,y) * g(x,y)\} = F(u,v) \cdot G(u,v)
\]

Spatial convolution (applying a filter kernel to every pixel) is equivalent to **pointwise multiplication** in the frequency domain. The FFT-based filtering pipeline is therefore:

1. **FFT** of image → \(F(u,v)\)
2. **FFT** of filter kernel → \(H(u,v)\)
3. **Pointwise multiply** → \(G(u,v) = F(u,v) \cdot H(u,v)\)
4. **Inverse FFT** → processed image \(g(x,y)\)

For large kernels, this is **vastly faster** than direct spatial convolution. [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)

## Circular vs. Linear Convolution (Wrap-Around)

Because the DFT assumes periodicity, the Convolution Theorem gives **circular convolution** — pixels that exit one edge wrap around and bleed into the opposite edge (ghosting artifact):

\[
(f \circledast g)(x,y) = \sum_{m,n} f(m,n)\, g\!\left((x-m)_M,\, (y-n)_N\right)
\]

**Fix:** **Zero-pad** the image by appending a border of zeros before computing the FFT. The padding provides a buffer zone so the periodic copies don't interact with the image. [geeksforgeeks](https://www.geeksforgeeks.org/computer-vision/fast-fourier-transform-in-image-processing/)

***

# Part 7: Computational Complexity — DFT vs. FFT

## Why Naïve DFT Is Expensive

For an \(N \times N\) image, the brute-force DFT computes \(N^2\) frequency points, each requiring a sum over all \(N^2\) pixels → total cost is **\(O(N^4)\)**. [geeksforgeeks](https://www.geeksforgeeks.org/computer-vision/fast-fourier-transform-in-image-processing/)

## The FFT Algorithm

The **Fast Fourier Transform** exploits two key insights: [homepages.inf.ed.ac](https://homepages.inf.ed.ac.uk/rbf/HIPR2/fourier.htm)

1. **Separability**: The 2D DFT can be decomposed into 1D DFTs along rows, then columns
2. **Divide-and-conquer**: Each 1D DFT of length \(N\) is split into even- and odd-indexed sub-problems, recursively:

\[
X_k = \underbrace{E_k}_{\text{Even-indexed terms}} + \underbrace{e^{-j2\pi k/N}}_{\text{Twiddle factor}} \cdot \underbrace{O_k}_{\text{Odd-indexed terms}}
\]

The **twiddle factor** \(e^{-j2\pi k/N}\) is periodic and symmetric — computed values are reused across multiple \(k\). This reduces 1D DFT cost from \(O(N^2)\) to \(O(N \log N)\).

**Total 2D FFT cost:**

\[
\underbrace{2N}_{\text{row + col passes}} \times \underbrace{O(N \log N)}_{\text{per 1D FFT}} = O(N^2 \log N)
\]

This is a dramatic improvement — for a \(1024 \times 1024\) image, \(O(N^4)\) ≈ \(10^{12}\) ops versus \(O(N^2 \log N)\) ≈ \(10^7\) ops. [homepages.inf.ed.ac](https://homepages.inf.ed.ac.uk/rbf/HIPR2/fourier.htm)

***

# Part 8: Reading the Frequency Spectrum

## What the Spectrum of an Image Tells You

| Feature in Spectrum | Meaning in Image |
|---|---|
| Bright center (DC component) | High mean brightness; dominant low-frequency structure |
| Diffuse cloud near center | Smooth regions, natural image with mostly coarse features |
| Oriented streaks at angle \(\theta\) | Strong edges/structures oriented at angle \(\theta\) in the image |
| Elevated outer regions | High-frequency noise or fine texture |
| Nearly uniform spectrum | Heavily corrupted image; noise overwhelms structure |
| Two bright dots on horizontal axis | Vertical stripes (sinusoid in x-direction) |
| Two bright dots on vertical axis | Horizontal stripes (sinusoid in y-direction) |

This follows from the FT of a pure sinusoid: \(\mathfrak{F}\{\sin(2\pi At)\} = \frac{1}{2i}[\delta(f-A) - \delta(f+A)]\), giving spectral peaks exactly at \(\pm A\). [micro.magnet.fsu](https://micro.magnet.fsu.edu/primer/java/digitalimaging/processing/fouriertransform/index.html)

## Low-pass vs. High-pass Filters in Frequency Domain

- **Gaussian blur (low-pass)**: Its spectrum is a **smooth blob centered at DC** — it passes low frequencies and attenuates high ones, which is why it blurs edges and removes noise [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)
- **Edge detection (high-pass)**: Its spectrum is **zero at DC and grows outward** — it suppresses smooth regions and amplifies abrupt changes (edges, texture) [mathworks](https://www.mathworks.com/help/images/fourier-transform.html)