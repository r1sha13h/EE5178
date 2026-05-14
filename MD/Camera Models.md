# 📷 Camera Models — Complete & Comprehensive Notes


## 1. Why Camera Models Exist

### Bare Sensor Problem

A raw digital sensor (CCD/CMOS) placed directly in front of a scene produces a **completely useless image**. Every scene point emits light in all directions, so every point activates every pixel simultaneously — resulting in a uniform, blurry mess.

> ❌ All scene points contribute to all sensor pixels → meaningless image

### Solution: The Pinhole Camera

A **pinhole** is literally just a tiny hole in a barrier placed in front of the sensor, inside a light-tight box. Every scene point reflects light rays in all directions — without any barrier, all those rays hit the entire sensor. The pinhole only lets through **one ray per scene point** (the ray aimed directly at the hole), which lands at exactly one spot on the sensor.

```
[Scene]  →  [tiny hole / aperture]  →  [image plane / sensor]
                    ↑
               the "pinhole"
```

**Result:** a scaled and inverted copy of the real world — a **perfect perspective projection**.

**Key terms:**
- **Aperture** — the tiny hole in the barrier
- **Image plane** — the flat surface where light rays land to form the image
- **Sensor plane** — the physical CCD/CMOS chip. In a digital camera, the image plane and sensor plane are the **same physical surface**. In film cameras, it's the film stock. Mathematically, it's an abstract flat plane at distance $f$ from the camera center.
- **Camera center** (center of projection) — the pinhole itself
- **Focal length $f$** — distance between the aperture and the image plane

> ⚠️ Subtle distinction: the **focal plane** is where light from a specific subject distance comes into sharp focus. The **image plane** is the fixed sensor surface. When focused correctly, these coincide. When not, the image is blurry.

**Key properties of a pinhole camera:**
- **Infinite depth of field** — near and far objects equally sharp (no lens to cause defocus)
- **Perfect perspective projection** — mathematically clean central projection
- **Very dark images** — only one ray per point gets through → lenses were invented to fix this

### Effect of Focal Length and Aperture Size

| Change | Effect |
|---|---|
| Halve focal length ($0.5f$) | Object projection is half the size |
| Increase aperture diameter | Multiple rays per point → image becomes blurrier |
| Ideal aperture | Infinitesimally small — perfectly sharp, but very dark |

### Solution to Blur: The Lens Camera

A lens **concentrates a whole bundle of rays** from a single scene point into a single sensor point, solving the brightness-vs-sharpness tradeoff of the pinhole.

Both models can be described with the **same math** under three conditions:
1. Use only **central rays** (the ray through the optical center — behaves identically in both)
2. Assume the lens camera is **in focus**
3. Assume the **focus distance $D'$ ≈ focal length $f$** of the pinhole

> ⚠️ **Focal length terminology:**
> - **Pinhole:** distance between aperture and sensor
> - **Lens:** distance where parallel rays converge
> - **In this course:** $f$ always refers to the aperture-sensor distance (pinhole definition)

***

## 2. The Camera as a Coordinate Transformation

A camera maps **3D world points → 2D image points**:

$$
\mathbf{x} = \mathbf{P}\mathbf{X}
$$

In homogeneous coordinates:

$$
\underbrace{\begin{bmatrix} x \\ y \\ w \end{bmatrix}}_{3\times1 \text{ image}} = \underbrace{\begin{bmatrix} p_1 & p_2 & p_3 & p_4 \\ p_5 & p_6 & p_7 & p_8 \\ p_9 & p_{10} & p_{11} & p_{12} \end{bmatrix}}_{3\times4 \text{ camera matrix } \mathbf{P}} \underbrace{\begin{bmatrix} X \\ Y \\ Z \\ 1 \end{bmatrix}}_{4\times1 \text{ world point}}
$$

- $(X, Y, Z)$ — real-world 3D coordinates of a scene point
- The extra $1$ is the homogeneous coordinate (always 1 for a real point; enables translation as matrix multiply — no physical meaning on its own)
- Output $(x, y, w)$ is **not yet a pixel** — divide by $w$ to get the actual pixel:

$$
\text{pixel} = \left(\frac{x}{w},\ \frac{y}{w}\right)
$$

The division by $w$ (= depth $Z$) is the **perspective divide** — this is precisely where "far objects look smaller" comes from.

### Why Only 11 DoF, Not 12?

Multiplying the entire matrix $\mathbf{P}$ by any scalar $\lambda \neq 0$ gives the same projection:

$$
(\lambda \mathbf{P}) X_w = \begin{bmatrix} \lambda x \\ \lambda y \\ \lambda w \end{bmatrix} \implies \text{pixel} = \left(\frac{\lambda x}{\lambda w}, \frac{\lambda y}{\lambda w}\right) = \left(\frac{x}{w}, \frac{y}{w}\right) \checkmark
$$

So overall scale carries no information → $12 - 1 = \mathbf{11}$ **DoF**. (Same reason a homography has $9 - 1 = 8$ DoF.)

***

## 3. Three Coordinate Systems

In any real camera setup, three distinct coordinate systems must be connected:

| System | Symbol | Meaning |
|---|---|---|
| **World coordinates** | $X_w$ | Global 3D frame where scene points are defined |
| **Camera coordinates** | $X_c$ | 3D frame relative to the camera center |
| **Image / Pixel coordinates** | $\mathbf{x}$ | 2D frame on the sensor (pixel addresses) |

### Understanding the Image Coordinate System

The sensor is a **rectangular grid of pixels**. Pixel $(0, 0)$ is at the **top-left corner** of the sensor — like a spreadsheet starting at cell A1:

```
(0,0) ──────────────► u (columns)
  │  pixel pixel pixel
  │  pixel pixel pixel
  │  pixel pixel pixel
  ▼
  v (rows)               (W-1, H-1) ← bottom-right
```

This is the **image coordinate system** — used by code, NumPy, OpenCV, etc.

### The Principal Point $(p_x, p_y)$

The **optical axis** (z-axis, pointing straight out of the lens) pierces the image plane at one point — the **principal point**. In an ideal camera this lands at the **image center** $= (W/2,\ H/2)$.

So there are **two origins on the same physical sensor**:

| Origin | Location | Used by |
|---|---|---|
| Camera coordinate origin | Principal point $(p_x, p_y)$ — center of sensor | Projection math |
| Image coordinate origin | Top-left corner $(0, 0)$ | Pixel addressing in code |

The offset $(p_x, p_y)$ bridges both systems. The projection equation **with** this shift:

$$
u_{\text{pixel}} = f \cdot \frac{X}{Z} + p_x, \qquad v_{\text{pixel}} = f \cdot \frac{Y}{Z} + p_y
$$

**Ideal vs. reality:**

| Situation | Principal Point |
|---|---|
| Ideal camera | Exactly at image center $(W/2, H/2)$ |
| Real camera | Slightly off-center due to manufacturing tolerances |

Camera calibration is needed to find the true $(p_x, p_y)$.

***

## 4. Deriving the Pinhole Camera Matrix

### Rearranged Pinhole (Image Plane in Front)

The image plane is placed **in front of** the camera center (not behind) for cleaner math — avoids image inversion. Using **similar triangles**:

$$
(X, Y, Z) \longrightarrow \left(\frac{X}{Z},\ \frac{Y}{Z}\right)
$$

### Case 1: $f = 1$, shared origins — Simplest Perspective Projection

$$
\mathbf{P} = [\mathbf{I}|\mathbf{0}] = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \end{bmatrix}
$$

### Case 2: Arbitrary focal length $f$

Points now map as $(X,Y,Z) \to (fX/Z,\ fY/Z)$:

$$
\mathbf{P} = \begin{bmatrix} f & 0 & 0 & 0 \\ 0 & f & 0 & 0 \\ 0 & 0 & 1 & 0 \end{bmatrix}
$$

### Case 3: Different camera and image origins — Adding $p_x, p_y$

$$
\mathbf{P} = \begin{bmatrix} f & 0 & p_x & 0 \\ 0 & f & p_y & 0 \\ 0 & 0 & 1 & 0 \end{bmatrix}
$$

***

## 5. Camera Matrix Decomposition

The camera matrix factors into **two parts**:

$$
\mathbf{P} = \underbrace{\begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix}}_{\mathbf{K} \text{ — intrinsics } (3\times3)} \underbrace{\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \end{bmatrix}}_{[\mathbf{I}|\mathbf{0}] \text{ — perspective projection } (3\times4)} = \mathbf{K}[\mathbf{I}|\mathbf{0}]
$$

| Factor | Size | Role |
|---|---|---|
| $[\mathbf{I}\|\mathbf{0}]$ | 3×4 | 3D → 2D perspective projection (assumes $z=1$ plane, shared origins) |
| $\mathbf{K}$ | 3×3 | 2D → 2D: accounts for focal length $f$ and origin shift $(p_x, p_y)$ |

> This form only handles cameras where the **world and camera are already aligned**. For the general case, a world-to-camera transform is needed.

***

## 6. World-to-Camera Transformation (Extrinsics)

### The Geometric Picture

The camera sits somewhere in the world with its own orientation. Every 3D world point must first be expressed in the **camera's frame**. Two steps:

1. **Translate** — subtract the camera center $\tilde{C}$ (camera position in world coordinates):
$$
\tilde{X}_w - \tilde{C}
$$
2. **Rotate** — apply $R$ to reorient world axes to camera axes:
$$
\tilde{X}_c = R(\tilde{X}_w - \tilde{C})
$$

### Explicit Matrix Form

$$
R(\tilde{X}_w - \tilde{C}) = \begin{bmatrix} r_{11} & r_{12} & r_{13} \\ r_{21} & r_{22} & r_{23} \\ r_{31} & r_{32} & r_{33} \end{bmatrix} \begin{bmatrix} X_w - C_x \\ Y_w - C_y \\ Z_w - C_z \end{bmatrix}
$$

Distributing $R$:

$$
\tilde{X}_c = R\tilde{X}_w - R\tilde{C} = R\tilde{X}_w + \mathbf{t}, \quad \mathbf{t} := -R\tilde{C}.
$$

### From Vector Form to Matrix Form

**Step 1 — vector form.**

$$
\tilde X_c \;=\; R\,\tilde X_w + \mathbf{t}.
$$

**Step 2 — pack $\mathbf{t}$ as a fourth column ⇒ $3\times 4$ extrinsic.**

$$
\tilde X_c \;=\; \bigl[\,R \mid \mathbf{t}\,\bigr] \begin{bmatrix} \tilde X_w \\ 1 \end{bmatrix}.
$$

**Step 3 — append $[\mathbf{0}^\top \mid 1]$ ⇒ square $4\times 4$ rigid transform.**

$$
\begin{bmatrix} \tilde X_c \\ 1 \end{bmatrix} \;=\; \underbrace{\begin{bmatrix} R & \mathbf{t} \\ \mathbf{0}^\top & 1 \end{bmatrix}}_{\text{extrinsic } 4\times 4} \begin{bmatrix} \tilde X_w \\ 1 \end{bmatrix}, \qquad \mathbf{t} = -R\tilde C.
$$

**Block structure explained:**

| Block | Value | Role |
|---|---|---|
| Top-left (3×3) | $R$ | Rotates the point |
| Top-right (3×1) | $-RC$ | Translates (in camera frame) |
| Bottom-left (1×3) | $\mathbf{0}^\top$ | Keeps it a point, not a direction |
| Bottom-right (1×1) | $1$ | Preserves homogeneous coordinate |

The last row $[\mathbf{0}^\top\ 1]$ is the standard rigid body transform structure (same as in 2D homographies with $[0\ 0\ 1]$ bottom row) — it ensures the output homogeneous coordinate remains 1.

### What Exactly is $\mathbf{t} = -RC$?

> ⚠️ **Common confusion:** $\mathbf{t} = -RC$ is **not** the camera center in camera-frame coordinates (that is always $(0,0,0)$ by definition — it's the origin of the camera frame).

$\mathbf{t} = -RC$ is the **world origin expressed in camera coordinates**. Plugging $X_w = (0,0,0)$ into the transform:
$$
X_c = R(0 - C) = -RC = \mathbf{t}
$$

| Symbol | Answers the question |
|---|---|
| $C$ | "Standing at the world origin — where is the camera?" |
| $\mathbf{t} = -RC$ | "Sitting at the camera — where does the world origin appear to be?" |

***

## 7. The Full Camera Pipeline

### Step-by-Step

| Step | From → To | Operation | Formula |
|---|---|---|---|
| 1 | World → Camera | Rotate + Translate | $X_c = R(X_w - C)$ |
| 2 | Camera → Normalized image | Perspective divide | $(X_c/Z_c,\ Y_c/Z_c)$ — far=small, near=large |
| 3 | Normalized → Pixels | Focal length + origin shift | $\begin{bmatrix}u\\v\\1\end{bmatrix} = K\begin{bmatrix}x\\y\\1\end{bmatrix}$ |

### Full Projection Formula

$$
\mathbf{P} \;=\; \underbrace{\begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix}}_{\substack{\text{intrinsics } (3\times 3) \\ \text{image} \to \text{image}}} \; \underbrace{[\,\mathbf{I} \mid \mathbf{0}\,]}_{\substack{\text{perspective projection } (3\times 4) \\ \text{camera} \to \text{image}}} \; \underbrace{\begin{bmatrix} R & -R\tilde C \\ \mathbf{0}^\top & 1 \end{bmatrix}}_{\substack{\text{extrinsics } (4\times 4) \\ \text{world} \to \text{camera}}}
$$

**Fusing projection and extrinsics.** The middle and right factors multiply cleanly:

$$
\mathbf{P} \;=\; \mathbf{K}\;\underbrace{[\,\mathbf{I} \mid \mathbf{0}\,]\begin{bmatrix} R & -R\tilde C \\ \mathbf{0}^\top & 1 \end{bmatrix}}_{=\,[\,R \,\mid\, -R\tilde C\,]\,=\,[\,R \,\mid\, \mathbf{t}\,]} \;=\; \mathbf{K}\,[\,R \mid \mathbf{t}\,], \qquad \mathbf{t} = -R\tilde C.
$$

Factoring $R$ out of $[\,R \mid -R\tilde C\,] = R\,[\,\mathbf{I} \mid -\tilde C\,]$ gives the **center-explicit form**:

$$
\mathbf{P} \;=\; \mathbf{K}\,R\,[\,\mathbf{I} \mid -\tilde C\,].
$$

Reading right-to-left, a world point passes through three stages:

- **Extrinsics** — corresponds to camera externals, world to camera transformation.
- **Perspective projection** — camera to image transformation, projecting 3D to the image plane.
- **Intrinsics** — corresponds to camera internals.

This collapses the three-matrix chain to the **clean two-matrix form**:

$$
\mathbf{P} \;=\; \underbrace{\begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix}}_{\mathbf{K}\ (3\times 3)} \; \underbrace{\begin{bmatrix} r_1 & r_2 & r_3 & t_1 \\ r_4 & r_5 & r_6 & t_2 \\ r_7 & r_8 & r_9 & t_3 \end{bmatrix}}_{[\mathbf{R}\mid\mathbf{t}]\ (3\times 4)} \;=\; \mathbf{K}\,[\,R \mid \mathbf{t}\,].
$$

***

## 8. General Pinhole Camera Matrix — Two Final Forms

$$
\mathbf{P} = \mathbf{KR}[\mathbf{I}|-\mathbf{C}] \quad \text{(translate first, then rotate)}
$$
$$
\mathbf{P} = \mathbf{K}[\mathbf{R}|\mathbf{t}] \quad \text{where } \mathbf{t} = -\mathbf{RC} \quad \text{(rotate first, then translate)}
$$

| Symbol | Type | Meaning |
|---|---|---|
| $\mathbf{K}$ | 3×3 intrinsics | Focal length + principal point |
| $\mathbf{R}$ | 3×3 rotation | Camera orientation in the world |
| $\mathbf{C}$ | 3×1 | Camera center in world coordinates |
| $\mathbf{t} = -RC$ | 3×1 | World origin in camera coordinates |
| $\mathbf{P}$ | 3×4 | Full camera projection matrix |

***

## 9. Degrees of Freedom (DoF)

### What is DoF?

$$
\text{DoF} = \text{total parameters} - \text{independent constraints}
$$

**Quick examples:**
- Point on a line → 1 DoF
- Point on a plane $(x,y)$ → 2 DoF
- Point in 3D $(x,y,z)$ → 3 DoF
- Point on unit circle $x^2 + y^2 = 1$: fix $x$, $y$ is determined → **1 DoF**

***

### Extrinsic Parameters: 6 DoF

**Translation: 3 DoF**
Camera position $(C_x, C_y, C_z)$ — three freely chosen numbers, one per axis.

**Rotation: 3 DoF**

A 3×3 rotation matrix has 9 entries, but must obey **orthonormality constraints** because a rotation physically:

1. **Preserves lengths** — a stick of 5cm stays 5cm → each column must be a **unit vector** → 3 constraints
2. **Preserves angles** — x, y, z axes stay perpendicular → columns must be **mutually perpendicular** → 3 constraints

If any column had length ≠ 1, the rotation would be stretching space (rotation + scaling). If columns weren't perpendicular, the axes would "squeeze together" — that's a shear, not a rotation.

Formally: $R^TR = I$ and $\det(R) = +1$ (the +1 distinguishes rotation from reflection).

$$
\text{Rotation DoF} = 9 - 3_{\text{unit length}} - 3_{\text{perpendicularity}} = 3
$$

The 3 rotation DoF correspond to: **Pitch** (tilt up/down), **Yaw** (turn left/right), **Roll** (tilt sideways).

$$
\text{Extrinsic DoF} = 3_{\text{translation}} + 3_{\text{rotation}} = \boxed{6}
$$

> 💡 Physically: to fully describe **where a camera is and where it points**, you need exactly 6 numbers — 3 for position, 3 for orientation.

***

### Intrinsic Parameters: Depends on Camera Type

#### Standard Camera — Square Pixels, No Skew

$$
\mathbf{K} = \begin{bmatrix} f & 0 & p_x \\ 0 & f & p_y \\ 0 & 0 & 1 \end{bmatrix} \quad \text{Free params: } f, p_x, p_y \Rightarrow \textbf{3 DoF}
$$

#### CCD Camera — Non-Square Pixels ($\alpha_x \neq \alpha_y$)

Real CCD sensors can have **rectangular pixels** — wider than tall or vice versa (historically common in analog broadcast cameras). One unit in x ≠ one unit in y, so you need **two separate focal lengths**:

- $\alpha_x = f \cdot k_x$ — focal length scaled by pixel width
- $\alpha_y = f \cdot k_y$ — focal length scaled by pixel height

```
Square pixel:        Rectangular pixel (CCD):
┌──┐                 ┌────┐
│  │                 │    │  ← wider than tall
└──┘                 └────┘
αx = αy = f          αx ≠ αy
```

$$
\mathbf{K} = \begin{bmatrix} \alpha_x & 0 & p_x \\ 0 & \alpha_y & p_y \\ 0 & 0 & 1 \end{bmatrix} \quad \text{Free params: } \alpha_x, \alpha_y, p_x, p_y \Rightarrow \textbf{4 DoF}
$$

#### Finite Projective Camera — Skewed Sensor

In an ideal camera, x and y pixel axes are perfectly perpendicular (90°). Due to manufacturing defects, the sensor can be **slightly sheared** — axes no longer exactly 90°. A point's x-coordinate then bleeds into its y-coordinate, captured by skew parameter $s$:

```
Normal pixel grid:         Skewed pixel grid:
┌──┬──┬──┐                ╱╱╱╱╱╱╱
│  │  │  │               ╱╱╱╱╱╱╱╱
└──┴──┴──┘               ╱╱╱╱╱╱╱
  axes at 90°              axes not at 90° → s ≠ 0
```

$$
\mathbf{K} = \begin{bmatrix} \alpha_x & s & p_x \\ 0 & \alpha_y & p_y \\ 0 & 0 & 1 \end{bmatrix} \quad \text{Free params: } \alpha_x, \alpha_y, s, p_x, p_y \Rightarrow \textbf{5 DoF}
$$

***

### Full DoF Summary

| Camera Type | Relaxed Assumption | Intrinsic DoF | + Extrinsic | Total DoF |
|---|---|---|---|---|
| **Standard** | — | 3 ($f, p_x, p_y$) | +6 | **9** |
| **CCD** | Non-square pixels | 4 ($\alpha_x, \alpha_y, p_x, p_y$) | +6 | **10** |
| **Finite Projective** | Skewed axes | 5 ($\alpha_x, \alpha_y, s, p_x, p_y$) | +6 | **11** |

Each row relaxes **one more physical assumption** about the sensor, adding one free parameter to $\mathbf{K}$.

> For reference: **Homography** (3×3) has $9 - 1 = 8$ DoF. **Camera matrix** (3×4) has $12 - 1 = 11$ DoF. The $-1$ always comes from homogeneous scale ambiguity.

***

## 10. The Big Picture Recap

$$
\underbrace{\mathbf{x}}_{\text{pixel}} = \underbrace{\mathbf{K}}_{\substack{\text{intrinsics} \\ 3\times3}} \underbrace{[\mathbf{R}|\mathbf{t}]}_{\substack{\text{extrinsics} \\ 3\times4}} \underbrace{X_w}_{\substack{\text{world} \\ \text{point}}}
$$

| Stage | Transformation | What it encodes |
|---|---|---|
| World → Camera | $[R\|-RC]$ (extrinsics) | Where the camera is and which way it faces |
| Camera → Normalized image | Perspective divide $X/Z,\ Y/Z$ | Depth-based foreshortening (far=small) |
| Normalized → Pixel | $\mathbf{K}$ (intrinsics) | Focal length, pixel shape, principal point |