Image processing:
- Blurring: removing high frequencies
- Sharpening: boosting high frequencies
- Low spatial frequency: smooth regions/overall shapes
- High spatial frequency: edges/details/noise

### Noise Removal Filters

1. Gaussian Noise
  - Box filter
  - Gaussian filter
2. Salt & Pepper Noise
  - Median filter
3. Shot Noise

### 3.2 2D Continuous Fourier Transform

$$F(u,v) = \iint_{-\infty}^{\infty} f(x,y)\, e^{-j 2\pi (ux + vy)}\, dx\, dy.$$

### 3.3 2D Discrete Fourier Transform

An image $f(x, y)$ of size $M \times N$ is transformed via

$$F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} f(x,y)\, e^{-j 2\pi \left(\tfrac{u x}{M} + \tfrac{v y}{N}\right)},$$

and inverted via

$$f(x,y) = \frac{1}{MN}\sum_{u=0}^{M-1}\sum_{v=0}^{N-1} F(u,v)\, e^{\,j 2\pi \left(\tfrac{u x}{M} + \tfrac{v y}{N}\right)}.$$

Why use frequency domain?
-  Anti-aliasing: remove high frequencies before downsampling to prevent moire patterns
- JPEG compression discards high frequencies
- Efficiency in convolution as it becomes pointwise multiplication in frequency domain
- Denoising

### Properties of 2D DFT:
- Linearity: $F\{af_1 + bf_2\} = aF\{f_1\} + bF\{f_2\}$
- Shift theorem: $F\{f(x-x_0, y-y_0)\} = F(u,v)e^{-j2\pi(ux_0/M + vy_0/N)}$
- Separability: 2D DFT can be computed as two 1D DFTs
- Shift: $F\{f(x-x_0, y-y_0)\} = F(u,v)e^{-j2\pi(ux_0/M + vy_0/N)}$
- Conjugate symmetry: half of the spectrum is redundant
- Rotation: rotation in spatial domain rotates spectrum by same angle
- Convolution theorem: $F\{f * g\} = F\{f\} \cdot F\{g\}$
- Differentiation: differentiation in spatial domain corresponds to multiplication by frequency - higher frequencies are amplified - high pass operation. $\frac{\partial f(x,y)}{\partial x} = j2\pi u \cdot F(u,v)$
- Periodicity: infinitely periodic spectrum in frequency domain.

Spectral Cross & Windowing fix.

### 7.1 Why the Naive DFT Is Expensive

For an $N \times N$ image, the brute-force DFT computes $N^2$ frequency points, each requiring a sum over all $N^2$ pixels:

$$\underbrace{N^2}_{\text{points}} \times \underbrace{N^2}_{\text{summations per point}} = O(N^4).$$

### 7.3 Total 2D FFT Cost

$$\underbrace{2N}_{\text{row + column passes}} \times \underbrace{O(N \log N)}_{\text{per 1D FFT}} = O(N^2 \log N).$$

For a $1024 \times 1024$ image, this is a dramatic improvement: $O(N^4) \approx 10^{12}$ operations versus $O(N^2 \log N) \approx 10^7$ operations.

### 8.3 What the Spectrum Tells You at a Glance

| Feature in spectrum         | Meaning in image                    |
| -----------------------------| -------------------------------------|
| Bright centre (DC)          | Bright image, mostly smooth.        |
| Diffuse cloud near centre   | Natural image with coarse features. |
| Streaks at angle $\theta$   | Edges oriented at angle $\theta$.   |
| Bright outer regions        | Fine detail/Edges or noise.         |
| Nearly uniform spectrum     | Image dominated by noise.           |
| Two dots on horizontal axis | Vertical stripes.                   |
| Two dots on vertical axis   | Horizontal stripes.                 |


### Low pass vs High pass filters in frequency domain

- **Gaussian blur (low-pass).** Its spectrum is a smooth blob centred at DC — it passes low frequencies (smooth regions) and attenuates high ones (fine details/edges), which is why it blurs edges and removes noise.
- **Edge detection (high-pass).** Its spectrum is zero at DC and grows outward — it suppresses smooth regions and amplifies abrupt changes (edges, texture).
