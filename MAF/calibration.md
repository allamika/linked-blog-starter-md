# Camera Calibration & Plane Position Estimation — Theory and Math

  

This document explains, in detail, the mathematics behind what this codebase

does: turning a single 2D camera into a device that can tell you the real-world

(X, Y) position, in millimeters, of anything lying on a known flat working

plane. It follows the actual pipeline implemented in

[calibration.py](calibration.py), [geometry.py](geometry.py) and

[validation.py](validation.py), stage by stage, deriving every formula used.

  

---

  

## 1. The problem in one picture

  

A camera turns 3D rays into 2D pixels. That projection is **not invertible**

in general: a single pixel corresponds to an entire ray in 3D space, not one

point. To recover a unique 3D position from a 2D pixel we need one extra

constraint. This project's constraint is: *the point of interest lies on a

known flat plane* (a table, a conveyor, a work surface). Given that

assumption, pixel → ray → ray-plane intersection → unique (X, Y) is

well-defined.

  

Getting there requires two calibration stages plus one coordinate-mapping

step:

  

| Stage | What is found | How |

|---|---|---|

| **A — Intrinsics** | The camera's internal optics: focal length, optical center, lens distortion | Many photos of a ChArUco board in different poses → `cv2.calibrateCamera` |

| **B — Plane pose** | Where the working plane sits relative to the camera (rotation + translation) | One photo of the board lying flat on the plane → `cv2.solvePnP` |

| **Mapping** | Pixel ↔ plane (X, Y) | A closed-form homography built from A + B |

  

---

  

## 2. The pinhole camera model

  

### 2.1 Ideal (distortion-free) projection

  

A point in the camera's own 3D coordinate frame, `Pc = (Xc, Yc, Zc)`, projects

to a pixel `(u, v)` through the **pinhole model**:

  

```

        [ fx   0   cx ]   [ Xc ]

s · [u,v,1]^T = K · Pc,   K = [  0  fy   cy ] ,  Pc = [ Yc ]

        [ 0    0   1  ]   [ Zc ]

```

  

i.e.

  

```

u = fx · Xc/Zc + cx

v = fy · Yc/Zc + cy

```

  

`K` is the **camera matrix** (a.k.a. intrinsic matrix):

  

- `fx, fy` — focal length in pixel units (focal length in mm × pixel density). Two values because pixels need not be perfectly square.

- `cx, cy` — the principal point: where the optical axis pierces the image plane, in pixels (near, but not necessarily exactly at, the image center).

  

This is exactly `cv2.calibrateCamera`'s `camera_matrix` output, and it is

what [calibration.py:160-163](calibration.py#L160-L163) solves for.

  

### 2.2 Lens distortion

  

Real lenses bend light more than the ideal pinhole model predicts, especially

toward the edges of the image (barrel/pincushion distortion) and, to a lesser

extent, due to imperfect lens-sensor alignment (tangential distortion).

OpenCV's default model corrects a *normalized* point `(x, y) = (Xc/Zc, Yc/Zc)`

before applying `K`:

  

```

r² = x² + y²

  

x_distorted = x · (1 + k1·r² + k2·r⁴ + k3·r⁶) + 2·p1·x·y + p2·(r² + 2x²)

y_distorted = y · (1 + k1·r² + k2·r⁴ + k3·r⁶) + p1·(r² + 2y²) + 2·p2·x·y

```

  

- `k1, k2, k3` — radial distortion coefficients (barrel/pincushion).

- `p1, p2` — tangential distortion coefficients (lens misalignment).

  

These five numbers are the `dist_coeffs` produced alongside `K`. The full

forward model is: 3D point → normalize → distort → multiply by `K` → pixel.

Going the other way (pixel → normalized undistorted ray) requires inverting

this nonlinear map — done iteratively by `cv2.undistortPoints`, used in

[geometry.py:28](geometry.py#L28).

  

### 2.3 Extrinsics: where things are relative to the camera

  

A 3D point known in some **object/world frame** `Pw` (e.g. a corner on the

calibration board, expressed in the board's own coordinate system) is related

to the same point in the **camera frame** `Pc` by a rigid transform:

  

```

Pc = R · Pw + t

```

  

- `R` — a 3×3 rotation matrix (the object's orientation relative to the camera).

- `t` — a 3×1 translation vector (the object's position relative to the camera).

  

OpenCV represents `R` compactly as a **Rodrigues rotation vector** `rvec`

(3 numbers: axis × angle), convertible to/from the full matrix with

`cv2.Rodrigues`. Together `(rvec, tvec)` are the *pose* — this is exactly

what `cv2.solvePnP` and `cv2.calibrateCamera` return per image

(`_rvecs`, `_tvecs` in [calibration.py:160-163](calibration.py#L160-L163),

`rvec, tvec` in [calibration.py:191](calibration.py#L191)).

  

Putting it together, the full forward projection of a known 3D object point

into a pixel is:

  

```

Pc = R·Pw + t                     (extrinsics: object → camera frame)

(x, y) = (Xc/Zc, Yc/Zc)           (perspective divide)

(x_d, y_d) = distort(x, y)        (lens distortion)

(u, v) = K · (x_d, y_d, 1)        (intrinsics: camera frame → pixel)

```

  

This is precisely `cv2.projectPoints`, used for on-screen overlays in

[gui.py:306-310](gui.py#L306-L310).

  

---

  

## 3. Why a ChArUco board

  

To solve for `K`, `dist_coeffs` (Stage A) or `(R, t)` (Stage B), the

calibration algorithm needs many 3D↔2D point correspondences: points whose

position is *known exactly* in some object frame, and whose pixel location

can be *detected exactly* in the image.

  

A **ChArUco board** (chessboard + embedded ArUco markers,

[calibration.py:34-41](calibration.py#L34-L41)) is the standard tool for

this:

  

- The **chessboard corners** (where black/white squares meet) can be

  localized to sub-pixel precision using image gradients — far more

  precisely than the markers themselves.

- The **ArUco markers** are individually identifiable (each has a unique

  binary ID), which lets the detector figure out *which* chessboard corner

  is which even if the board is only partially visible, rotated, or tilted —

  a plain chessboard is ambiguous under partial occlusion or symmetry.

- Board geometry (`squares_x`, `squares_y`, `square_length_m`,

  `marker_length_m` in [config.py:11-22](config.py#L11-L22)) fully determines

  every internal corner's 3D position in the board's own flat coordinate

  frame: `(col·square_length, row·square_length, 0)`. Because the board is

  rigid and flat, these coordinates are known to the precision with which the

  board was printed and measured — the single most important real-world

  error source in this whole system (see §8).

  

`cv2.aruco.CharucoDetector.detectBoard()` finds the markers, decodes their

IDs, and interpolates the interior chessboard corners, refining each to

sub-pixel accuracy. `board.matchImagePoints()` then pairs each detected

corner ID with its known 3D coordinate on the board, producing the

`(obj_pts, img_pts)` correspondence arrays that feed every calibration call

in this codebase.

  

---

  

## 4. Stage A — Intrinsic calibration

  

Implemented by `IntrinsicCalibrator` in

[calibration.py:117-163](calibration.py#L117-L163).

  

### 4.1 Data collection

  

For each captured frame, the board is detected and its corners matched to

known 3D board-frame coordinates: `(obj_pts, img_pts)` — one such pair

per detected corner, one array pair per captured frame. These accumulate

across many frames (`self._obj_points`, `self._img_points`), each frame

showing the board at a **different pose** (position/orientation/tilt/distance).

  

Multiple, varied poses are essential, not optional: a single view under-

constrains the problem (e.g. a frontal, un-tilted board cannot distinguish

some distortion parameters from focal length), and repeating the *same* pose

adds no new information. Tilting the board and covering the whole frame

(especially corners/edges, where distortion is strongest) is what makes the

solve well-conditioned.

  

### 4.2 The optimization: Zhang's method

  

`cv2.calibrateCamera` ([calibration.py:160-162](calibration.py#L160-L162))

implements Zhang's calibration method (Zhang, 2000), in two steps:

  

**Step 1 — Closed-form initialization.** For a planar object like a

chessboard, each view's correspondences define a **homography** `H`

(a 3×3 projective transform, undistorted-camera model) between board-plane

coordinates and image points. Stacking the orthogonality/equal-norm

constraints that `H = K·[r1 | r2 | t]` implies across all views yields a

linear system solved (via SVD) for the 5 intrinsic parameters and, per view,

an initial `(R, t)`. This gives a good starting point without needing

distortion coefficients yet.

  

**Step 2 — Nonlinear refinement.** All parameters (`K`, `dist_coeffs`, and

every view's `(rvec, tvec)`) are then jointly refined by nonlinear

least-squares (Levenberg-Marquardt) to minimize the total **reprojection

error**:

  

```

minimize   Σ_views Σ_corners  ‖ img_pt − project(obj_pt; K, dist_coeffs, rvec, tvec) ‖²

```

  

i.e. project every known board corner back into its image using the current

parameter estimate, and adjust the parameters to make the predicted pixel

match the actually-detected pixel as closely as possible, summed over every

corner of every captured frame simultaneously.

  

### 4.3 RMS reprojection error

  

The single number `cv2.calibrateCamera` returns first (`rms` in

`CalibrationResult`, [calibration.py:26-31](calibration.py#L26-L31)) is the

root-mean-square of those residuals, in pixels:

  

```

rms = sqrt( (1/N) · Σ ‖img_pt − project(obj_pt)‖² )

```

  

over all N corner observations across all captured frames. It is the single

best at-a-glance health check for Stage A: it says how consistently the fitted

`(K, dist_coeffs)` model, together with each frame's own fitted pose, explains

every corner that was actually detected. Values well under 1 pixel are

typical for a good webcam calibration with a reasonable number of varied

poses; values above ~1 px usually indicate too few/too-similar poses, a

poorly-printed board, or motion blur during capture.

  

Note what this number does *not* tell you: it says nothing about physical

(X, Y) accuracy in millimeters — that is only measured downstream, by the

validation stage (§7). A low RMS in pixels is necessary but not sufficient

for accurate real-world measurements, particularly if the printed board

geometry (§8) doesn't match what was configured.

  

### 4.4 Held-out split (optional)

  

`split_corner_ids` ([calibration.py:79-101](calibration.py#L79-L101)) can

split the board's internal corners into a left half and a right half (by

column). When enabled, `IntrinsicCalibrator.add_sample` is restricted to the

left half only (`calib_ids`), so the right half's corners never influence

`K`/`dist_coeffs` at all. This is the standard train/test split idea applied

to camera calibration: it lets the right half later serve as a genuinely

independent accuracy check (§7.2), rather than measuring the fit only on the

same data it was fit to.

  

---

  

## 5. Stage B — Plane pose estimation

  

Implemented by `estimate_plane_pose`

([calibration.py:166-194](calibration.py#L166-L194)).

  

Once `K` and `dist_coeffs` are known and fixed, the board is placed flat on

the physical working plane whose coordinate system is to be used from then

on — its position defines the world origin, and its two in-plane axes define

world X and Y. A single frame is captured, its corners detected and matched

to known board-frame 3D coordinates exactly as in Stage A, and

  

```

cv2.solvePnP(obj_pts, img_pts, K, dist_coeffs) → (rvec, tvec)

```

  

solves the **Perspective-n-Point (PnP)** problem: given ≥3 (here, several

dozen) 3D↔2D correspondences and known intrinsics, find the single rigid

transform `(R, t)` that makes `project(obj_pt; K, dist_coeffs, R, t)` best

match every observed `img_pt`, again by nonlinear least-squares (the same

reprojection-error minimization as §4.2, but now with `K`/`dist_coeffs` held

fixed and only `(rvec, tvec)` free).

  

The result **is** the plane's pose: `R` is the plane's orientation relative

to the camera, `t` is the plane's origin's position relative to the camera.

From here on, "world frame" and "plane frame" are the same thing: the flat

Z=0 plane defined by the board in this one captured pose. Everything the

system reports as an (X, Y) position is a coordinate in *this* frame,

established at the moment "Capture Plane Frame" is clicked — if the physical

plane or the board's placement on it changes, this stage must be redone.

  

If `calib_ids` (the held-out split) is active, Stage B is likewise restricted

to the left half only, leaving the right half completely untouched by

*both* stages — reserved purely for the accuracy check in §7.2.

  

---

  

## 6. Pixel ↔ plane mapping

  

This is the part that makes the system useful at runtime: converting an

arbitrary clicked pixel, or an automatically detected colored marker's pixel

centroid, into millimeters on the plane — without re-running PnP for every

query point.

  

### 6.1 Deriving the homography

  

Implemented in `compute_planar_homography`

([geometry.py:7-20](geometry.py#L7-L20)). Take any point that lies *on the

plane*, i.e. with plane-frame coordinates `Pw = (X, Y, 0)` — Z is always

exactly 0 because it's on the plane by definition. Applying the extrinsic

transform:

  

```

Pc = R·Pw + t = R · [X, Y, 0]^T + t

```

  

Writing `R = [r1 | r2 | r3]` (its three columns), the third column `r3`

multiplies the Z=0 term and drops out entirely:

  

```

Pc = X·r1 + Y·r2 + t

```

  

This is linear in `(X, Y, 1)`:

  

```

Pc = [ r1  r2  t ] · [X, Y, 1]^T

```

  

Projecting through the (undistorted) intrinsics `K`:

  

```

s·[u, v, 1]^T = K·Pc = K·[r1 | r2 | t] · [X, Y, 1]^T = H · [X, Y, 1]^T

```

  

So the composite 3×3 matrix

  

```

H = K · [ r1 | r2 | t ]

```

  

is a **homography**: a single linear (projective) map that takes any

homogeneous plane-frame point `(X, Y, 1)` directly to its homogeneous

*undistorted* pixel `(u, v, 1)` (up to the scale factor `s`, removed by

dividing through by the third component). It packages "project through the

plane's known pose" as one 3×3 matrix, avoiding a full 3D ray-plane

intersection for every point.

  

Physically, `H` is equivalent to intersecting the camera ray through

`(u, v)` with the plane defined by point `t` (a point on the plane) and

normal `R·[0,0,1]^T` (the plane's own Z axis, expressed in camera-frame

coordinates) — but as one matrix multiply instead of an explicit

ray/plane-intersection computation.

  

### 6.2 Going backward: pixel → plane

  

Implemented in `pixel_to_plane` ([geometry.py:23-31](geometry.py#L23-L31)).

`H` maps plane → pixel; to go the other way, invert it:

  

```

H⁻¹ · [u, v, 1]^T = s' · [X, Y, 1]^T   →   (X, Y) = (row0/row2, row1/row2) of the result

```

  

`homography_inv = H⁻¹` is precomputed once per plane calibration

([gui.py:639](gui.py#L639), [gui.py:701-702](gui.py#L701-L702)) rather than

recomputed per query point.

  

One subtlety: `H` was derived from *undistorted* pixel coordinates (the

pinhole model in §2.1 has no distortion term). A raw pixel read off the live

frame is *distorted*. So before applying `H⁻¹`, the pixel must first be

undistorted:

  

```python

undistorted = cv2.undistortPoints(pt, K, dist_coeffs, P=K)   # distorted px → undistorted px (still in pixel units, via P=K)

world = H⁻¹ · [undistorted_x, undistorted_y, 1]^T

(X, Y) = world[:2] / world[2]

```

  

`cv2.undistortPoints` inverts the forward distortion model of §2.2

iteratively (it has no closed form) and, with `P=K` supplied, re-expresses

the result back in pixel units (rather than the default normalized camera

coordinates) so it's directly usable with `H`. This full pipeline — raw pixel

→ undistort → apply `H⁻¹` → normalize by third coordinate — is exactly

`pixel_to_plane`, and it is the single function that answers "where is this

pixel, in real-world mm, on my working plane?" for both manual clicks

([gui.py:466](gui.py#L466)) and the automatic color-marker tracker

([gui.py:381](gui.py#L381)).

  

### 6.3 Why this only works for points on the plane

  

`H` (and its inverse) is only valid for points that actually lie at Z=0 in

the plane frame. A pixel showing an object floating above the plane (say, a

marker on top of a 20mm-tall part) will still get mapped *somewhere* by

`pixel_to_plane` — but that answer is the (X, Y) where the camera ray

through that pixel crosses Z=0, not the true (X, Y) above the object itself.

The taller the object and the more oblique the camera angle, the larger this

parallax error becomes. This is a fundamental limitation of single-camera,

plane-constrained triangulation, not a bug — it's the price of needing only

one camera and one extra assumption (flatness) instead of two cameras and

explicit stereo triangulation.

  

---

  

## 7. Validation: measuring real-world accuracy

  

Implemented in [validation.py](validation.py). The RMS from Stage A is a

pixel-space fit-quality number; it doesn't directly answer "how many mm off

is a plane-position estimate in practice?" Two independent methods answer

that question, both included in the code because they measure slightly

different things.

  

### 7.1 Moved-board validation (`run_validation_capture`)

  

The idea: place the (already-calibrated) board anywhere else on the same

physical plane, capture a frame, and compare two *independent* estimates of

each corner's position:

  

1. **Ground truth**, via a fresh, independent `solvePnP` on this new frame

   (`rvec_v, tvec_v`) — this gives each board corner's position in the

   camera frame, `cam_pt = R_v · local_pt + t_v`, which is then converted

   into the *original* plane frame established in Stage B:

  

   ```

   world_pt = R_plane^T · (cam_pt − t_plane)

   ```

  

   (the inverse of the Stage-B rigid transform: since `R` is orthonormal,

   `R⁻¹ = R^T`). This yields `(x_gt, y_gt, z_gt)` for every corner —

   `z_gt` should be ≈0 if the board is indeed lying flat on the same plane

   used in Stage B; if it isn't, that's a real planarity violation, and the

   GUI flags it (`mean_abs_z_gt_mm`, [gui.py:740](gui.py#L740),

   `planarity_warn_mm` in [config.py:58](config.py#L58)).

  

2. **Prediction**, via the production code path: take that same corner's

   raw detected pixel `(u, v)` and run it through `pixel_to_plane` — the

   exact function used at runtime for arbitrary marker/click positions.

  

The per-corner error is then simply the planar (X, Y) distance between the

two:

  

```

err = sqrt( (x_pred − x_gt)² + (y_pred − y_gt)² )

```

  

reported in mm. Aggregated per capture as `mean_err_mm`, `rmse_mm`

(root-mean-square, more sensitive to outliers than the mean), and

`max_err_mm`.

  

This method is powerful because the ground truth for a *newly-placed* board

is derived independently of `pixel_to_plane`/`H` — it comes from a fresh

PnP solve, not from the homography path being tested — so it isn't

circular. Its only requirement is that the board must be genuinely flat and

resting on the plane at capture time (any tilt/lift shows up as nonzero

`z_gt`, which would otherwise silently corrupt the accuracy numbers).

  

### 7.2 Held-out-half validation (`measure_held_out_accuracy`)

  

The alternative, using the split-board setup from §4.4/§5: with the board

still in the *exact* placement used for Stage B, the right (held-out) half's

corners have known ground-truth positions *for free* — they're just

`board.matchImagePoints`'s object points, `(X, Y, 0)` in the plane frame,

because that half's corners are rigidly attached to the very board that

defined the plane frame in the first place. No repositioning, no second

`solvePnP` needed:

  

```

err = sqrt( (x_pred − X)² + (y_pred − Y)² )

```

  

where `(x_pred, y_pred) = pixel_to_plane(u, v, ...)` and `(X, Y)` is the

corner's known board-geometry coordinate.

  

This is the cleaner statistical test in the classical sense (train/test

split: `calib_ids` fit `K`/`dist_coeffs`/plane pose, `validate_ids` never

touch any of that), but it's more fragile operationally — it's only valid

as long as the board has not moved even slightly since "Capture Plane

Frame" was clicked, since it relies on "this half's local frame == the

plane frame" being exactly true.

  

Both methods log every corner to a CSV

(`log_validation_csv`, [validation.py:168-178](validation.py#L168-L178)) for

offline inspection, and the GUI displays running aggregates.

  

---

  

## 8. Where real-world error actually comes from

  

Given the math above, four factors dominate final (X, Y) accuracy:

  

1. **Board manufacturing accuracy.** Every 3D "known" coordinate fed into

   every solve in this codebase comes from `BoardConfig.square_length_m`

   times an integer row/column — i.e., it *assumes* the printed board is

   geometrically perfect. If a printer scales the pattern (e.g. printer

   "fit to page" is on), every downstream number is scaled by the same

   factor. `save_board_image`'s docstring

   ([calibration.py:58-62](calibration.py#L58-L62)) and the GUI's export

   dialog ([gui.py:709-719](gui.py#L709-L719)) both call this out

   explicitly: print at 100% scale and verify a printed square with

   calipers before trusting anything downstream.

2. **Pose diversity during Stage A.** Per §4.1, too few or too-similar

   board poses under-constrain `K`/`dist_coeffs`, inflating both the

   reported RMS and, more importantly, real-world error at image regions

   not well-sampled during capture (typically the corners/edges of the

   frame, where distortion is strongest).

3. **Corner-detection precision.** Sub-pixel accuracy of ChArUco corner

   detection depends on image sharpness (focus, lack of motion blur),

   lighting evenness, and print contrast/quality.

4. **Planarity of the target.** As noted in §6.3, the whole pixel→plane

   mapping assumes Z=0. Any real deviation of the object being measured

   from the calibrated plane translates directly into position error

   proportional to both that height and the camera's obliqueness at that

   pixel.

  

The validation stage (§7) is the empirical way to catch all four at once,

in the same physical units (mm) the system is meant to report — rather than

trusting the Stage A RMS (in pixels) as a proxy for something it doesn't

actually measure.

  

---

  

## 9. End-to-end summary

  

```

Stage A (many poses)                 Stage B (one pose, board flat on plane)

 detect ChArUco corners               detect ChArUco corners

 → (obj_pts, img_pts) per frame       → (obj_pts, img_pts)

 → cv2.calibrateCamera                → cv2.solvePnP(..., K, dist)  [K, dist fixed]

 → K, dist_coeffs, RMS                → rvec, tvec  (plane pose)

                                              │

                                              ▼

                                 H = K · [r1 | r2 | t]   (geometry.compute_planar_homography)

                                              │

                                              ▼

                                       H_inv = H⁻¹  (precomputed once)

  

Runtime, per query pixel (u, v):

  undistort (u, v) with (K, dist_coeffs)

  → apply H_inv, normalize by 3rd coord

  → (X, Y) in meters on the plane        (geometry.pixel_to_plane)

  

Validation:

  independent solvePnP on a new board placement, or a held-out half of the

  same placement, gives ground-truth (X, Y) to compare pixel_to_plane's

  output against, in mm.

```

  

Every arrow above corresponds to one function in this codebase; none of it

is approximate beyond the inherent least-squares fits and the four real-world

error sources in §8.