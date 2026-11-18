# Image Enhancement for Remote Sensing

## Description
This project trains a **U-Net** to restore degraded Landsat 8 red-band (B4) surface reflectance and tests whether the improvement **holds on data the model has not seen**: a different region of the same scene, a different date, and both. It also tests whether restoration helps detect **small changes** (60–240 m) between two acquisitions.

The project is inspired by **Mahmoud Afifi's work on learnable camera ISPs** and applies similar principles to remote sensing imagery.

## Revision (November 2026)
The first version of `ISPs.ipynb` (March 2025, see git history) trained and evaluated the network on the **same single image**, using the clean image as both the training target and the evaluation reference. That design measures memorisation, not enhancement. It also used a plain encoder–decoder without skip connections, normalised each image by its own minimum and maximum, had no baseline, and assessed change detection only visually.

The revised notebook fixes these issues:

| Element | Revised protocol |
|---|---|
| Data | Landsat 8 Collection 2 Level-2 SR, WRS-2 path 196 / row 24, 12 May 2025 and 9 March 2025 (Microsoft Planetary Computer) |
| Scaling | Official SR scale factors; fixed 0–0.3 reflectance range for every image |
| Masking | QA_PIXEL fill, cloud, cirrus, shadow and snow pixels excluded |
| Split | Train and validate in the west of the May scene; test on the east of the May scene (spatial hold-out), the west of the March scene (temporal hold-out) and the east of the March scene (both); 6 km buffers |
| Model | U-Net with skip connections, residual output, L1 loss |
| Baselines | Degraded input, Gaussian and median filters tuned on validation data |
| Metrics | PSNR, SSIM, bootstrap 95 % CIs, paired Wilcoxon tests; change-detection F1 and IoU |

## Results
500 test patches (64 × 64 px) per test set. Full tables in `results/`.

**Restoration (SSIM)**

| Test set | Degraded input | Best filter (Gaussian) | U-Net |
|---|---|---|---|
| Training region (for reference) | 0.756 | 0.742 | 0.873 |
| Spatial hold-out | 0.757 | 0.744 | **0.859** |
| Temporal hold-out | 0.759 | 0.746 | **0.838** |
| Spatial + temporal hold-out | 0.761 | 0.748 | **0.839** |

The U-Net beat the best filter on 100 % of held-out patches (Wilcoxon p < 1e-80). Its gain shrank on the unseen date (SSIM 0.873 on training patches vs 0.838–0.839 in March), so part of the training-region score does not transfer.

**Small-area change detection (F1)**

| Test set | Degraded input | Gaussian filter | U-Net |
|---|---|---|---|
| Spatial hold-out | 0.784 | **0.787** | 0.608 |
| Temporal hold-out | 0.806 | **0.810** | 0.723 |
| Spatial + temporal hold-out | 0.804 | **0.807** | 0.723 |

Better restoration scores did **not** translate into better detection of small changes: differencing the U-Net outputs detected small changes less accurately than differencing the degraded images. A likely explanation is that the network, trained to remove noise, also suppresses small real changes. Image-quality metrics alone are therefore not a sufficient test of whether enhancement helps a downstream task.

## Limitations
- The degradation (noise, blur, 2× resampling) is simulated; real sensor artefacts differ.
- One band, one path/row and two dates; transfer to other regions, seasons and sensors is untested.
- Synthetic rectangular changes; real land-cover change is more varied.
- Small model trained on CPU.

## Tools and Technologies
Python, PyTorch, Rasterio, scikit-image, SciPy, pandas, Matplotlib, Jupyter.

## Installation
1. **Clone the repository**:
   ```bash
   git clone https://github.com/Samuelyohanna9/image-enhancement-remote-sensing.git
   cd image-enhancement-remote-sensing
   ```
2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```
3. **Run the notebook** (downloads the Landsat windows automatically; training takes about 30 minutes on a CPU):
   ```bash
   jupyter notebook ISPs.ipynb
   ```
