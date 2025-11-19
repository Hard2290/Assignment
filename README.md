# **Focus Detection and Efficiency Analysis**

This project implements a complete end-to-end pipeline for:

- PPG signal cleaning using Adaptive LMS filtering

- Extraction of heart-band and breath-band signals

- Detection of heart peaks, Inter-Beat Intervals (IBI), HR, HRV

- Detection of breathing peaks and respiration rate

- A custom-designed focus efficiency scoring system

- Detection of focus intervals

- Generation of plots and summary tables for reporting

This repository contains the Jupyter notebook (.ipynb) implementing the entire pipeline.

## **1. How to Run the Notebook**

### **Option A: Using Google Colab**

- Upload the notebook **(Assignment.ipynb)** to Google Colab.

- Upload the dataset as **raw_data.csv** to Colab’s file system.

- Run all cells in order.

- All visualizations will appear inline.

### **Option B: Running Locally**

- Install **Python 3.8+**

- Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate
```
- Install dependencies:
```bash
pip install -r requirements.txt
```
- Place the dataset as:
```bash
raw_data.csv
```
- Run the notebook in **Jupyter**:
```bash
jupyter notebook Assignment.ipynb
```

## **2. Dependencies**

The following Python libraries are required:
| Library          | Purpose                        |
| ---------------- | ------------------------------ |
| **numpy**        | Numerical operations           |
| **pandas**       | Data handling, rolling windows |
| **scipy.signal** | Filtering, peak detection      |
| **plotly**       | Interactive visualizations     |
| **typing**       | Type hints                     |

we may install all at once:
```bash
pip install numpy pandas scipy plotly
```

## **3. External Data or Models Used**
✔ **External Data**

The project uses a single dataset:
```bash
raw_data.csv
```
It must contain the following columns:

- timestamp

- green (raw PPG)

- acc_mag (accelerometer magnitude)

- gyro_mag (gyroscope magnitude)

If individual axis components exist (acc_x, acc_y, acc_z, …), the code automatically computes magnitudes.

✔ **External Models**

No external ML models are used.
All computations rely on:

- Adaptive LMS filtering

- Butterworth band-pass filtering

- Statistical thresholding

- Custom physiological scoring formulas

## **4. Known Limitations / Issues**
### **a. Uneven timestamp spacing**

Timestamps in the dataset may not be uniformly spaced, resulting in: 

- Focus interval rectangles appearing visually “thick” or “thin”

- Focus score plots having irregular spacing

This is only a visualization artefact, not a data issue.

### **b. Heart Rate appears flat**

HR is computed only at detected peaks → very slow updates.

### **c. HRV & Respiration rate look flat**

Used windowed medians → naturally change slowly.

### **d. Sensitivity to motion**

If movement is too high, HR & respiratory extraction may degrade.
This can reduce focus intervals.

### **e. LMS Filter Stability**

Adaptive filters can overshoot if:

- noise/PPG magnitudes change rapidly

- gains (acc_gain, gyro_gain) explode

However, the implementation includes strong normalization to prevent divergence.

## **5. Key Visualizations**
### **a. Raw vs Cleaned PPG Signal**

Shows removal of motion artifacts using adaptive LMS filter.

![Image](https://github.com/user-attachments/assets/9cfe5d50-bdc5-4e29-afd3-f5614eca9011)

### **b. Heart-Band & Breath-Band PPG Signals**

Demonstrates band-pass filtering into heart and respiration frequency ranges.

![Image](https://github.com/user-attachments/assets/574b424c-60c9-4ab6-a322-15acd7e33565)

### **c. Detected Heart Peaks & Breath Peaks**

Shows peak detection in filtered signals.

![Image](https://github.com/user-attachments/assets/197ed7f0-01af-4cb4-be32-bfddb8340332)

### **d. Physiological Metrics Over Time**

Visualizes:

ACC & GYRO motion

Heart Rate

HRV (RMSSD)

Respiration Rate

<img width="1170" height="900" alt="Image" src="https://github.com/user-attachments/assets/b685412a-80eb-42a6-b079-7f56259f1645" />

### **e. Focus Intervals (Shaded Regions)**

Visual verification of focus regions detected by the algorithm.

<img width="1170" height="525" alt="Image" src="https://github.com/user-attachments/assets/59094a71-219e-4426-8e13-8b351070ec19" />

### **f. Summary Table of Focus Intervals**

Includes start time, end time, duration, and efficiency score.

<img width="1170" height="400" alt="Image" src="https://github.com/user-attachments/assets/297a9f31-3905-4599-ba4d-8b8f96ba2c8d" />

### **g. Focus Efficiency Score Over Time**

Shows how focus score evolves across the session.

<img width="1170" height="525" alt="Image" src="https://github.com/user-attachments/assets/0d683ee6-580f-4a68-a5a5-f21c87d2edfc" />

## **6. Focus Efficiency Scoring**

**Focus Intervals are identified based on**:

- Low Movement - ACC & GYRO medians must be below an adaptive threshold between Q2–Q3.

- Steady Heart Rate - HR Coefficient of Variation <= 10%.

- Acceptable HRV - HRV >= median HRV of session (too low HRV → stress; too high HRV → drowsiness or sleep).

- Respiratory Stability:
    - Breathing rate between 10–20 bpm

    - Respiratory variability (RRV) ≤ 3 bpm

    - Breathing amplitude smoothness ≤ 5%

**Focus Efficiency Scoring (0–100)**:

The score combines motion + physiology + respiration:

| Component               | Weight | Description                                               |
| ----------------------- | ------ | --------------------------------------------------------- |
| **Motion Score**        | 50%    | Measures stillness. High motion → low score.              |
| **Physiological Score** | 30%    | 50% each of HRV (higher HRV-calmer and more focused) and HR stability (smaller fluctuation-more focus).        |
| **Respiratory Score**   | 20%    | 40% Breathing rate, 30% Low RRV and 30% Smooth amplitude. |

Final score:

**Score = 0.5 x MotionScore + 0.3 x PhysScore + 0.2 x RespScore**
