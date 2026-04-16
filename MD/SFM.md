***

## Structure from Motion (SfM) — Complete Slide Notes

> **Note:** Slides 1–12 and several others contain only images or videos with no text content. All extractable text from every slide is compiled below, organized by slide/page reference.

***

## Slide 13 — Credits
*(Image-heavy slide)*
- **SfM slides credits: Prof. Noah Snavely**

***

## Slide 14 — Problem Statement
*(Image-heavy slide)*
- **Given a set of corresponding points in two or more images, compute the camera parameters and the 3D point coordinates**

***

## Slides 15–16 — Visual Slides
*(Images only — no text content)*

***

## Slide 17 — Overview
- **2D image captures**
- **COLMAP SfM construction**
*(Contains two supporting images)*

***

## Slide 18 — Overall Pipeline (COLMAP)
- **Overall Pipeline** *(linked to the COLMAP paper: https://demuc.de/papers/schoenberger2016sfm.pdf)*
*(Contains a pipeline diagram image)*

***

## Slide 19 — Section Divider: Correspondence Search
- **1] Correspondence search**
*(Contains a supporting image)*

***

## Slide 20 — Feature Extraction (Header Slide)
- **Feature Extraction** *(section header only)*

***

## Slides 21–22 — Feature Extraction (Detail)
- **Feature Extraction**
*(Each slide contains a supporting image)*

***

## Slide 23 — Feature Matching (Header Slide)
- **Feature Matching** *(section header only)*

***

## Slide 24 — Feature Matching (Detail)
*(Image only — no text content)*

***

## Slide 25 — Geometric Verification (Header Slide)
- **Geometric Verification** *(section header only)*

***

## Slides 26–27 — Geometric Verification (Detail)
- **Geometric Verification**
*(Each slide contains a supporting image)*

***

## Slide 28 — Section Divider: Incremental Reconstruction
- **2] Incremental Reconstruction**
*(Contains a supporting image)*

***

## Slide 29 — Initialization (Header Slide)
- **Initialization** *(section header only)*

***

## Slide 30 — Initialization (Detail)
*(Image only — no text content)*

***

## Slide 31 — Image Registration (Header Slide)
- **Image Registration** *(section header only)*

***

## Slide 32 — Image Registration (Detail)
- **Objective:** Register new images to the existing model by solving the **Perspective-n-Point (PnP)** problem.
- **Method:** Uses **2D-3D correspondences** (matching image features to previously triangulated 3D points).
- **Estimation Goals:**
  - **Pose Estimation (P_c):** Determining the position and orientation of the new camera.
  - **Intrinsic Parameters:** Estimated simultaneously for uncalibrated cameras.
- **Solution:** Employs RANSAC to mitigate the effects of **outlier-contaminated** correspondences.

***

## Slide 33 — Next Keyframe Problem
- **Next Keyframe problem**
*(Contains a supporting image)*

***

## Slide 34 — Triangulation (Header Slide)
- **Triangulation** *(section header only)*

***

## Slide 35 — Triangulation (Detail)
*(Image only — no text content)*

***

## Slide 36 — Bundle Adjustment (Header Slide)
- **Bundle Adjustment** *(section header only)*

***

## Slide 37 — Bundle Adjustment (Detail)
- **w_ij = binary value**
- **d(.) = Euclidean (L2) norm**
*(Contains a supporting image with the BA cost function)*

***

## Slide 38 — Incremental Reconstruction (Summary)
- **Incremental Reconstruction (summary)**
  - Initialize motion from two images using fundamental matrix
  - Initialize structure by triangulation
  - For each additional view:
    - Determine projection matrix of new camera using all the known 3D points that are visible in its image — calibration
    - Refine and extend structure: compute new 3D points, re-optimize existing points that are also seen by this camera — triangulation
  - Refine structure and motion: bundle adjustment

***

## Slide 39 — Incremental Reconstruction in COLMAP
- **Incremental Reconstruction in COLMAP**
*(Contains a supporting diagram image)*

***

## Slide 40 — Visual Slide
*(Image only — no text content)*

***

## Slide 41 — Structure from Motion (Section Header)
- **Structure from Motion**
*(Contains a supporting image)*

***

## Slide 42 — SfM: Variable Breakdown
**The Breakdown of Variables:**
- **m:** Number of cameras (views).
- **n:** Number of 3D points.
- **2mn (Total Knowns):** Each of the *n* points is observed in *m* images, and each 2D observation (x, y) provides 2 independent equations.

**The Breakdown of Unknowns:**
- **Camera Parameters (11m):** Each 3×4 projection matrix has 12 elements. Since scale is arbitrary in projective space, each camera has **11 degrees of freedom (DOF)**.
- **3D Points (3n):** Each 3D point (X, Y, Z) has **3 DOF**.
- **Projective Ambiguity (−15):** The entire reconstruction is only recoverable up to a 4×4 projective transformation. A 4×4 matrix has 16 elements, but 1 is lost to scale, leaving **15 DOF** that we cannot solve for without a fixed reference frame.

**Solvability Constraint — Knowns vs. Unknowns:**

To solve for both structure and motion, the number of independent equations must meet or exceed the number of free variables:

\[ 2mn \geq 11m + 3n - 15 \]

- **2mn:** Observations (2 per point per image).
- **11m:** Unknown camera parameters (11 DOF per camera).
- **3n:** Unknown 3D point coordinates (3 DOF per point).
- **15:** Subtraction of the shared **Projective Ambiguity** (the 4×4 coordinate transformation).

***

## Slides 43–44 — Visual Slides
*(Images only — no text content)*

***

## Slide 45 — Projective Ambiguity
*(Contains a supporting image)*
- **Q:** This represents a **4×4 full-rank matrix** (a transformation). Think of this as a "warping" function that can **rotate**, scale, translate, or even shear the entire 3D space.

***

## Slide 46 — Projective Ambiguity Takeaway
*(Contains a supporting image)*
- **The takeaway:** Since the 2D image looks the same in both cases, an algorithm cannot tell if it is looking at the original scene or a warped scene viewed by a warped camera.

***

## Slide 47 — Affine Ambiguity
*(Contains a supporting image)*
- **A:** This is the "distorter." Since it is a general full-rank matrix, it can perform **non-uniform scaling** and **shearing**.
- **t:** A 3×1 **translation vector** representing the position.

***

## Slide 48 — Visualizing the Ambiguity
- **Visualizing the Ambiguity**
- **What is preserved:** If two lines are parallel in the actual 3D scene, they will remain parallel in an affine reconstruction.
- **What is lost:** The "squarishness" of objects. For example, a cube might be reconstructed as a parallelepiped (a slanted box). You cannot tell if an angle is truly 90° or if one side is exactly twice as long as another unless they are pointing in the same direction.
*(Contains a supporting image)*

***

## Slide 49 — Similarity / Metric Ambiguity
*(Contains a supporting image)*
- **s:** Scale factor
- **R:** Rotation matrix
- **t:** A 3×1 translation vector representing the position.

***

## Slide 50 — What is Preserved (Similarity)
*(Contains a supporting image)*
**What is Preserved?**
- **Angles are correct:** A 90° corner in the real world looks like a 90° corner in your model.
- **Relative distances are correct:** If one wall is twice as long as another in real life, it will be twice as long in your model.
- **Parallelism is preserved:** Parallel lines stay parallel.

***

## Slide 51 — Comparison of Ambiguities

| **Ambiguity Type** | **Constraints needed** | **What is preserved?** |
|:---:|:---:|:---:|
| Projective | None (just matches) | Straight lines |
| Similarity | Parallelism | Parallel lines, ratios of lengths on parallel lines |
| Affine | Orthogonality / Calibration | Angles, ratios of all lengths (True shape) |

***

## Slide 52 — COLMAP Substitutions
*(Contains a supporting image)*
**Substitutions for COLMAP:**
- https://colmap.github.io/
- https://lpanaf.github.io/eccv24_glomap/
- https://vggsfm.github.io/
- https://vgg-t.github.io/

***

## Slide 53 — Thank You
- **THANK YOU!**

***

## Slide 54 — References
1. Structure from Motion (COLMAP) Paper: https://demuc.de/papers/schoenberger2016sfm.pdf
2. https://www.cs.unc.edu/~ronisen/teaching/spring_2023/web_materials/lecture_16_sfm.pdf
3. https://saurabhg.web.illinois.edu/teaching/ece549/sp2020/slides/lec17_sfm.pdf

***

## Slides 55–57 — Global SfM & Appendix
- **Global SFM:** https://lpanaf.github.io/eccv24_glomap/
*(Slides 55, 56, 57 each contain supporting images with no additional extractable text)*