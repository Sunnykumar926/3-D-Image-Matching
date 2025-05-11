# 📷 Camera Models and Distortion in COLMAP

## 🔧 What is Focal Length?

In COLMAP, `focal` refers to **focal length in pixels**. It plays a crucial role in projecting 3D points to 2D by controlling how "zoomed in" the camera is.

COLMAP uses focal length and other intrinsic parameters to accurately **reconstruct 3D scenes**.

---

## 📐 Camera Model Parameters

Different camera models in COLMAP expect different parameter sets:

| Model Name       | Model ID | Parameters                                 |
|------------------|----------|---------------------------------------------|
| `simple-pinhole` | 0        | `[f, cx, cy]`                               |
| `pinhole`        | 1        | `[fx, fy, cx, cy]`                          |
| `simple-radial`  | 2        | `[f, cx, cy, k]`                            |
| `opencv`         | 4        | `[fx, fy, cx, cy, k1, k2, p1, p2]`          |

- `f`, `fx`, `fy`: focal lengths  
- `cx`, `cy`: principal point (image center)  
- `k`, `k1`, `k2`: **radial distortion coefficients**  
- `p1`, `p2`: **tangential distortion coefficients**

To disable distortion, set coefficients to `0`. For example:
### 📐 Mathematical Model

Let a pixel be at position `(x, y)` relative to the center `(cx, cy)`.

#### Normalize the distance from the center:

`r² = x² + y²`

#### Apply radial distortion:

- `x_distorted = x * (1 + k * r²)`
- `y_distorted = y * (1 + k * r²)`

This simulates how the pixels are distorted based on their distance from the center and the distortion coefficient `k`.

---

To simulate an **ideal lens** (no distortion), use the following parameters:


param_array = np.array([fx, fy, cx, cy, 0., 0., 0., 0.])  # Ideal lens (no distortion)

## 🔍 Understanding Radial Distortion (`k` in `simple-radial`)

Radial distortion bends straight lines due to lens curvature:

- **Barrel distortion** (`k > 0`): lines bend outward.
- **Pincushion distortion** (`k < 0`): lines bend inward.

---

### 📐 Mathematical Model

Let a pixel be at position \((x, y)\) relative to the center \((cx, cy)\):

1. **Normalize the distance** from the center:
   \[
   r^2 = x^2 + y^2
   \]

2. **Apply radial distortion**:
   \[
   x_{\text{distorted}} = x \cdot (1 + k \cdot r^2) \\
   y_{\text{distorted}} = y \cdot (1 + k \cdot r^2)
   \]

---

### 📌 Example with `k = 1`

Point: `(x, y) = (0.5, 0.5)`

Compute radial distance:

`r² = 0.5² + 0.5² = 0.25 + 0.25 = 0.5`

Apply distortion:

- `x_distorted = 0.5 * (1 + 1 * 0.5) = 0.5 * 1.5 = 0.75`
- `y_distorted = 0.5 * (1 + 1 * 0.5) = 0.5 * 1.5 = 0.75`

➡️ This pushes the pixel **away from the center**, simulating a **"bulging" image**.

---

## 🔍 What About the `opencv` Model?

- `[k1, k2]`: **Radial distortion** coefficients  
- `[p1, p2]`: **Tangential distortion** coefficients (model lens shift or tilt)

These coefficients allow **precise modeling of real-world lens imperfections**.

---

## ✅ Why This Matters

COLMAP uses these parameters to:

- ✅ Accurately **reconstruct 3D geometry**
- 🔄 **Undistort** images for better feature matching
- 🎯 Simulate realistic **camera projection** and image formation

```python
