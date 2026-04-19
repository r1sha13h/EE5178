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
$$F(u,v) = F(u + k_1 M,\;v + k_2 N)$$
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
Because DFT assumes periodicity, the Convolution Theorem performs **Circular Convolution** by default.

**Problem:** Pixels that "exit" the right side wrap around and bleed into the left side.

**Fix:** **Zero-pad** the image with a border of zeros to achieve **Linear Convolution** (standard filtering). The padding provides a buffer zone preventing interaction with periodic copies.

---

## Slide 26 — Inverse 2D DFT
To reconstruct spatial image f(x, y) from frequency coefficients F(u, v):

$$f(x,y) = \frac{1}{NM}\sum_{k=0}^{N-1}\sum_{l=0}^{M-1} \hat{h}(k,l)\, e^{i(\omega_k x + \omega_l y)}$$

---

## Slide 27 — Differentiation Property
Differentiating in the spatial domain → multiplying by frequency variable in the frequency domain:

$$\frac{\partial f(x,y)}{\partial x} \iff j2\pi u \cdot F(u,v)$$

*Spatial derivative is finding the rate of change is pixel intensity and is equivalent to multiplication by ramp function in frequency domain.*

---

## Slides 28–31 — Fast Fourier Transform (FFT)

### Fast Fourier Transform (FFT)
The 2D Fast Fourier Transform (FFT) is an optimized algorithm for computing the Discrete Fourier Transform (DFT). It treats a 2D grid as a series of separable 1D transforms, exploiting the symmetry and periodicity of complex exponentials to eliminate redundant calculations.

### Computational Cost of Naïve DFT
- One frequency point (u, v) requires summing all N×N pixels → **O(N⁴)** for full 2D DFT

### 2D FFT via Row-Column Decomposition
Using the **Separability** property:
1. Apply 1D FFT to each row: O(N log N) per row, N rows → O(N² log N)
2. Apply 1D FFT to each column of result

Total: **O(N² log N)** instead of O(N⁴)

### 1D FFT Algorithm
- **Divide and Conquer:** splits data into **even** and **odd** samples, solves as smaller sub-problems, recombines
- Reduces complexity from O(N²) to **O(N log N)**
- Efficiency relies on symmetry and periodicity of the **"Twiddle Factor"** $e^{-i2\pi kn/N}$, allowing reuse of previously computed values

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
