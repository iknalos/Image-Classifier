# 🔬 Hyperspectral Image Classifier

A web application for non-destructive sample classification using hyperspectral imaging and machine learning. Built for snapshot mosaic cameras with custom spectral filter arrays, it allows researchers and producers to identify sample types from raw TIFF images — no lab analysis required.

[![Streamlit App](https://static.streamlit.io/badges/streamlit_badge_black_white.svg)](https://iknalos-image-classifier.streamlit.app)

---

## 📌 Overview

Traditional sample classification often relies on chemical or physical analysis — expensive, destructive, and time-consuming. This application leverages **hyperspectral imaging** to capture the unique spectral signature of each sample across multiple wavelength bands, then uses a trained machine learning model to classify unknown samples in seconds.

The pipeline is fully guided — from uploading training images to receiving a prediction — with no coding required.

---

## ✨ Features

- **4-step guided workflow** — upload, ROI selection, training, prediction
- **Dual ROI selector** — interactive sliders to select two analysis regions on the sample
- **Patch-based feature extraction** — divides each ROI into sub-patches for richer training data
- **Ensemble classifier** — SVM + Random Forest + XGBoost soft-voting, proven in peer-reviewed hyperspectral imaging research
- **Leave-One-Image-Out validation** — honest accuracy with no data leakage
- **Spectral visualisation** — signature plots, distance heatmap, PCA cluster view, confusion matrix
- **Batch prediction** — upload a ZIP of unknown images and classify all at once
- **Google Drive integration** — load training data or prediction ZIPs directly from Drive
- **Model export** — download the trained model as `.pkl` and reload it anytime
- **File management** — add or remove training files individually from the sidebar

---

## 🔬 How It Works

### Camera & Filter
The app is designed for snapshot mosaic cameras fitted with a custom **N×N Fabry-Pérot mosaic filter**. Each raw TIFF captured contains spectral channels interleaved in a repeating tile pattern across the sensor. The app demosaics this raw image to extract each band separately.

### Pipeline

```
Raw TIFF  →  Demosaic (N² bands)  →  ROI extraction  →  Patch features  →  Classifier  →  Prediction
```

1. **Demosaicing** — separates the mosaic tile pattern into individual spectral band images
2. **ROI selection** — user defines two regions of interest on the sample
3. **Patch extraction** — each ROI is divided into 30×30 px sub-patches; each patch produces a feature vector of normalised band means + band standard deviations
4. **Training** — ensemble classifier trained with Leave-One-Image-Out cross-validation to prevent data leakage
5. **Prediction** — patches from new images are classified individually; final prediction is the average confidence across all patches

---

## 🧠 Classifier

The app uses a **soft-voting ensemble** of three models:

| Model | Role |
|-------|------|
| **SVM (RBF kernel)** | Best single-model performer for high-dimensional spectral data |
| **Random Forest** | Handles non-linear band interactions; resistant to overfitting |
| **XGBoost** | Gradient boosting; captures patterns the other two may miss |

Each model outputs a confidence probability per class. The final prediction is the weighted average of all three votes.

> **Why not deep learning?**
> CNN and transformer models achieve higher accuracy on large hyperspectral datasets but require thousands of labelled samples. For small laboratory datasets (6–20 images per class), the ensemble approach consistently outperforms deep learning while training in seconds rather than hours.

**Research backing:**
- *ScienceDirect (2024)* — EBM-SVM showed exceptional stability in hyperspectral sample classification
- *PubMed* — SVM and MLP achieved F1 scores up to 0.99 in hyperspectral imaging studies
- *MDPI Remote Sensing (2024)* — SVM and Random Forest among top performers for sample grading with HSI

---

## 🚀 Getting Started

### Run locally

```bash
git clone https://github.com/iknalos/Image-Classifier.git
cd Image-Classifier

pip install -r requirements.txt

streamlit run app.py
```

Opens at `http://localhost:8501`

### Deploy on Streamlit Cloud

1. Fork this repository
2. Go to [share.streamlit.io](https://share.streamlit.io)
3. Connect your GitHub account
4. Select this repo → `main` → `app.py`
5. Click **Deploy**

---

## 📁 File Naming Convention

Training TIFF filenames must contain the class label somewhere in the name. The label is detected automatically — no manual tagging needed.

**Example:**
```
ClassA_sample_1.tiff
ClassB_100K_2.tiff
ClassC_capture_3.tiff
```

Supported built-in labels can be extended by editing the `get_label()` function in `app.py`.

---

## 📦 Project Structure

```
Image-Classifier/
├── app.py               # Main Streamlit application
└── requirements.txt     # Python dependencies
```

---

## 🛠️ Requirements

```
streamlit
tifffile
numpy
scikit-learn
matplotlib
scipy
joblib
xgboost
google-auth
google-api-python-client
requests
```

Python 3.9+ recommended.

---

## 📷 Camera Setup

| Parameter | Value |
|-----------|-------|
| Recommended camera | Snapshot mosaic hyperspectral camera |
| Filter type | N×N Fabry-Pérot mosaic array |
| Image format | TIFF (16-bit, uint16) |
| Recommended exposure | Consistent across all captures |

> **Important:** All training and prediction images should be captured under the same exposure settings and lighting conditions. Exposure differences between sessions will reduce classification accuracy.

---

## 📊 Validation

The app uses **Leave-One-Image-Out (LOIO) cross-validation** — the most honest validation strategy for small datasets. For each fold, all patches from one image are held out as the test set while the model trains on all remaining images. This prevents data leakage that would occur if patches from the same image appeared in both training and test sets.

---

## 🔮 Future Improvements

- [ ] Automatic tile size detection from raw TIFF
- [ ] Wavelength calibration — map band indices to actual nm values
- [ ] Support for configurable camera/filter setups
- [ ] Export predictions as CSV report
- [ ] Multi-user session support with persistent storage

---

## 📐 Data Pipeline Detail

### Step 2 — Dual ROI Selection
Two separate ROI boxes are drawn over the sample. These capture different spatial areas for more spatial diversity in training data.

### Step 3 — Patch Extraction
```
1 image × 2 ROIs × ~20 patches = ~40 samples per image
3 images × 40 patches           = ~120 samples per class
```
Each patch → 162-dimensional feature vector (band means + standard deviations), all from just a few raw files.

### Step 4 — Leave-One-Image-Out Validation
Trained on N-1 images, tested on 1, repeated N times. Patches from the same image never appear in both train and test — fully honest accuracy.

---

## 📄 License

MIT License — free to use, modify and distribute.

---

## 👤 Author

**iknalos**  
Built with [Streamlit](https://streamlit.io) · Powered by scikit-learn, XGBoost & tifffile
