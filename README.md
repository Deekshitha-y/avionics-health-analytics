# Avionics Health Analytics

## Overview

**Avionics Health Analytics** is an AI-based health monitoring system designed to assess the operational condition of avionics subsystems using flight telemetry data. The project combines statistical monitoring, unsupervised anomaly detection, and temporal degradation analysis to generate a unified health score (0–100) that supports avionics reuse assessment.

The system is inspired by challenges in **Reusable Launch Vehicles (RLVs)**, where avionics components must be evaluated after each mission to ensure safe and efficient reuse.

---

## Problem Statement

Traditional avionics requalification methods rely on manual inspection and fixed threshold checks. These approaches are effective for detecting major faults but often fail to identify subtle multi-sensor anomalies and gradual performance degradation over time.

This project proposes a **layered analytics framework** that analyzes multivariate telemetry data, monitors statistical boundaries, isolates anomalies, evaluates temporal variance degradation, and aggregates them into an interpretable health score for reuse decision support.

---

## Architecture & Methodology

The pipeline processes high-dimensional avionics telemetry across three distinct analytical layers:

```
Telemetry Logs (.npy)
       │
       ▼
┌─────────────────────────────────────────────────────────┐
│ Layer 1: Statistical Boundary Monitoring (3-Sigma Rule) │
└──────────────────────────┬──────────────────────────────┘
                           │
       ▼───────────────────┴──────────────────────────────┐
│ Layer 2: Multivariate Anomaly Detection (Isolation Forest)
└──────────────────────────┬──────────────────────────────┘
                           │
       ▼───────────────────┴──────────────────────────────┐
│ Layer 3: Temporal Degradation Analysis (Rolling Variance)
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
          ┌──────────────────────────────────┐
          │ Unified Health Score Aggregator  │
          │         (Score: 0 - 100)         │
          └──────────────────────────────────┘
```

### Layer 1 – Statistical Boundary Monitoring
Applies the **Three-Sigma Rule (\( \mu \pm 3\sigma \))** per sensor channel using training baselines to detect sensor readings that deviate significantly from nominal operating conditions.

### Layer 2 – Multivariate Anomaly Detection
Employs an **Isolation Forest** to detect contextual, non-linear anomalies within high-dimensional telemetry space across all sensor channels simultaneously.

### Layer 3 – Temporal Degradation Analysis
Uses **Rolling Variance analysis** across moving temporal windows to detect gradual sensor instability, signal drift, and mechanical/electrical wear over time.

### Health Score Aggregation
Combines normalized penalty outputs from all three layers into a single scalar health score ranging from **0 to 100**:
$$\text{Health Score} = 100 - (\alpha \cdot \text{Rule Penalty} + \beta \cdot \text{Anomaly Penalty} + \gamma \cdot \text{Degradation Penalty})$$

---

## Dataset Characteristics

The system uses multivariate avionics telemetry stored in NumPy (`.npy`) format:

* **164 telemetry log files**
* High-dimensional multivariate time-series data:
  * **25-sensor configuration** (~65.9% of fleet)
  * **55-sensor configuration** (~34.1% of fleet)
* **Data Split**: 50% Training Baseline, 50% Mission Testing Data
* **Data Matrix**:
  * Rows: Time steps (temporal readings)
  * Columns: Sensor channels (thermal, electrical, pressure, and stability indicators)

---

## Project Structure

```text
avionics-health-analytics/
├── data/
│   └── data/               # Raw and processed telemetry data (.npy files)
├── outputs/
│   ├── logs/               # Execution and analysis logs (analysis.log)
│   ├── metrics/            # Evaluation metrics, stats, and comparison CSVs
│   └── plots/              # Generated performance and evaluation charts
├── src/
│   ├── __init__.py
│   ├── config.py           # Paths and hyperparameters configuration
│   ├── load_data.py        # Telemetry ingestion and parsing routines
│   ├── preprocess.py       # Data scaling, normalization, and handling
│   ├── rule_based.py       # Layer 1: 3-Sigma threshold monitoring
│   ├── anomaly_model.py    # Layer 2: Isolation Forest anomaly detector
│   ├── degradation.py      # Layer 3: Rolling variance degradation scorer
│   ├── health_score.py     # Aggregation logic for 0–100 health score
│   ├── utils.py            # Logging setup, file helpers, directory utilities
│   └── main.py             # Pipeline execution script
├── tests/
│   └── test_logic.py       # Unit tests for health score bounds & algorithms
├── requirements.txt        # Python package dependencies
└── README.md               # Project documentation
```

---

## Getting Started

### Prerequisites
* Python 3.9+ (tested up to Python 3.13)
* `pip` package manager

### Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/shatakshi85/avionics-health-analytics.git
   cd avionics-health-analytics
   ```

2. **Create and activate a virtual environment:**
   ```bash
   # Windows (PowerShell)
   python -m venv venv
   .\venv\Scripts\Activate.ps1

   # Linux / macOS
   python3 -m venv venv
   source venv/bin/activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

---

## Running the Pipeline

Execute the end-to-end analytics workflow:

```bash
python src/main.py
```

This will:
1. Load telemetry files from `data/data/`.
2. Compute baseline statistics and fit the Isolation Forest model on nominal flight data.
3. Evaluate test missions across the three monitoring layers.
4. Calculate fleet health statistics.
5. Export detailed metric reports to `outputs/metrics/`.
6. Generate visualizations in `outputs/plots/`.

---

## Running Unit Tests

Run the test suite to verify scoring logic and boundary constraints:

```bash
python -m unittest discover tests
```
or with `pytest`:
```bash
pytest tests/
```

---

## Key Results & Outputs

* **Stable Anomaly Detection**: Average mission anomaly rate maintained between **0.5% – 2.0%**.
* **Configuration Invariance**: Robust detection behavior across both 25-sensor and 55-sensor configurations.
* **Nominal Fleet Health**: Fleet health scores predominantly fall within **95 – 100**, verifying nominal subsystem performance with zero fleet-wide critical failures.

Visualizations generated in `outputs/plots/`:
* `v2_health_distribution.png`: Histogram of overall subsystem health scores.
* `v2_health_trend.png`: Temporal progression of health scores across flight cycles.
* `v2_layer_contribution.png`: Breakdown of penalty contributions across the 3 layers.
* `v2_configuration_comparison.png`: Comparison between 25-sensor and 55-sensor configurations.
* `v2_contamination_sensitivity.png`: Sensitivity curve across Isolation Forest contamination rates.

---

## Technologies Used

* **Language**: Python
* **Data Processing**: NumPy, Pandas
* **Machine Learning**: Scikit-Learn (Isolation Forest)
* **Visualization**: Matplotlib
* **Testing**: Python Unittest / Pytest

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
