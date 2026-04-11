## Keypoint Detection and SIFT

$$ H = \begin{bmatrix} D_{xx} & D_{xy} \\ D_{yx} & D_{yy} \end{bmatrix} $$

$$ m(x,y) = \sqrt{(L(x+1,y)-L(x-1,y))^2 + (L(x,y+1)-L(x,y-1))^2} $$
$$ \theta(x,y) = \tan^{-1}\!\left(\frac{L(x,y+1)-L(x,y-1)}{L(x+1,y)-L(x-1,y)}\right) $$

$$ \theta^* = \text{argmax(histogram)} $$

$$ w(x,y) = \exp\!\left(-\frac{(x-x_k)^2+(y-y_k)^2}{2(1.5\sigma)^2}\right) $$

$$ \text{ratio} = \frac{d_{\text{nearest}}}{d_{\text{second nearest}}} $$

## 2D Transformations

| Transform | Matrix |
|---|---|
| Scale | $\begin{bmatrix} s_x & 0 \\ 0 & s_y \end{bmatrix}$ |
| Rotate | $\begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$ |
| Shear | $\begin{bmatrix} 1 & s_x \\ s_y & 1 \end{bmatrix}$ |
| Flip across y | $\begin{bmatrix} -1 & 0 \\ 0 & 1 \end{bmatrix}$ |
| Flip across x | $\begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$ |
| Flip across origin | $\begin{bmatrix} -1 & 0 \\ 0 & -1 \end{bmatrix}$ |
| Identity | $\begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}$ |

**Translation**:
$$\begin{bmatrix} 1 & 0 & t_x \\ 0 & 1 & t_y \\ 0 & 0 & 1 \end{bmatrix}$$

**Scaling**:
$$\begin{bmatrix} s_x & 0 & 0 \\ 0 & s_y & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

**Rotation**:
$$\begin{bmatrix} \cos\theta & -\sin\theta & 0 \\ \sin\theta & \cos\theta & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

**Shear (combined XY)**:
$$\begin{bmatrix} 1 & s_x & 0 \\ s_y & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

**Flip across y**:
$$\begin{bmatrix} -1 & 0 & 0 \\ 0 & 1 & 0 \\ 0 & 0 & 1 \end{bmatrix}$$

## Homography and RANSAC

$$A_i = \begin{bmatrix} -x & -y & -1 & 0 & 0 & 0 & x'x & x'y & x' \\ 0 & 0 & 0 & -x & -y & -1 & y'x & y'y & y' \end{bmatrix}$$

$$\text{error}_i = \|p'_i - H \cdot p_i\|_2$$

$$N \geq \frac{\log(1-p)}{\log(1-w^k)}$$

## Camera Models

$$u_{\text{pixel}} = f \cdot \frac{X}{Z} + p_x, \qquad v_{\text{pixel}} = f \cdot \frac{Y}{Z} + p_y$$

$$
\mathbf{P} = \underbrace{\begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix}}_{\mathbf{K} \text{ — intrinsics } (3\times3)} \underbrace{\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \end{bmatrix}}_{[\mathbf{I}|\mathbf{0}] \text{ — perspective projection } (3\times4)} = \mathbf{K}[\mathbf{I}|\mathbf{0}]
$$

$$
\begin{bmatrix} X_c \\ Y_c \\ Z_c \\ 1 \end{bmatrix} = \underbrace{\begin{bmatrix} \mathbf{R} & -\mathbf{RC} \\ \mathbf{0}^\top & 1 \end{bmatrix}}_{4\times4 \text{ extrinsic matrix}} \begin{bmatrix} X_w \\ Y_w \\ Z_w \\ 1 \end{bmatrix}
$$

$$
\mathbf{x} = \underbrace{\mathbf{K}}_{\text{intrinsics}} \underbrace{[\mathbf{I}|\mathbf{0}]}_{\text{projection}} \underbrace{\begin{bmatrix} R & -RC \\ \mathbf{0}^\top & 1 \end{bmatrix}}_{\text{extrinsics}} X_w
$$


$$
\mathbf{P} = \underbrace{\begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix}}_{\mathbf{K}\ (3\times3)} \underbrace{\begin{bmatrix} r_1 & r_2 & r_3 & t_1 \\ r_4 & r_5 & r_6 & t_2 \\ r_7 & r_8 & r_9 & t_3 \end{bmatrix}}_{[\mathbf{R}|\mathbf{t}]\ (3\times4)}
$$
