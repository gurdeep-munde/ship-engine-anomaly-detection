# Ship Engine Anomaly Detection — Industrial IoT Sensor Data

Unsupervised anomaly detection on continuous condition-monitoring data from a marine propulsion engine. Three approaches compared (IQR multivariate rule, One-Class SVM, Isolation Forest) and reconciled to produce a high-confidence shortlist of suspect cycles.

## The problem

Unplanned engine downtime on a vessel at sea is one of the most expensive failures in commercial shipping: lost cargo time, towage and salvage fees, port re-routing, and a real safety risk for the crew. Continuous condition-monitoring data is cheap to collect and modern engines emit plenty of it — but raw sensor traces are noisy, and only a small fraction of unusual readings reflect a real developing fault.

The operational question for a fleet operator is not "are there outliers?" but **"can we surface a 1–5% shortlist of cycles that a chief engineer should actually look at, with enough method-agreement that we are not flooding them with false positives?"** That framing — flag rate as a tunable parameter, method agreement as the trust signal — drives every choice in this analysis.

## The data

A real dataset of **19,535 cycles** with six channels continuously sampled from a ship engine:

| Channel | What it tells you when it drifts |
|---|---|
| Engine RPM | Sustained over-speed → wear / overheating; under-speed → fuel-delivery or mechanical issue |
| Lube oil pressure | Low → insufficient lubrication, friction, damage; high → blockage in oil delivery |
| Fuel pressure | High → injector or filter issue; low → fuel-pump issue, poor combustion |
| Coolant pressure | Low → coolant leak; high → blockage or head-gasket failure |
| Lube oil temperature | High → lubricant degradation; low → cold operation, inadequate film |
| Coolant temperature | High → overheating (thermostat / leak / flow); low → not yet at operating temperature |

The dataset is published openly by the Cambridge Data Science programme — see [`data/README.md`](data/README.md) for the URL and schema.

## What's in this project

| Notebook | What it covers | Time to run |
|---|---|---|
| [`ship_engine_anomaly_detection.ipynb`](notebooks/ship_engine_anomaly_detection.ipynb) | EDA → IQR rule → One-Class SVM with hyperparameter sweep → Isolation Forest → three-way method comparison and PCA visualisation | ~2 min on CPU |

A PDF version of the writeup is in [`ship_engine_anomaly_detection_report.pdf`](ship_engine_anomaly_detection_report.pdf) for skim-reading without opening the notebook.

## Headline results

| Method | Flag rate | Anomalies | Agreement with IF | Notes |
|---|---|---|---|---|
| **IQR (≥2 features flagged)** | 2.16% | 422 | 40% | Cheap, transparent, but single-feature thinking misses multivariate cases |
| **One-Class SVM** (RBF, γ=0.1, ν=0.05) | 5.01% | 978 | 89% | Tuned via grid search over γ × ν; PCA projection shows clean separation |
| **Isolation Forest** (500 trees, contamination=0.05) | 5.00% | 977 | — | Most consistent; insensitive to feature scaling; fastest at inference time |
| **Three-way intersection** | **1.2%** | **233** | — | The high-confidence shortlist a chief engineer should triage first |

## The operational framing

Three threads run through the notebook:

1. **Flag rate is a knob, not an output.** Every method here is tunable to land in whatever band a downstream operator can absorb (1–5% in this case). The numbers above came from explicit hyperparameter sweeps against a target flag rate, not from accepting defaults. That is the right discipline for production deployment.

2. **Method agreement matters more than method choice.** Isolation Forest and One-Class SVM agree on 89% of their flagged rows — strong evidence both are locking onto genuine multivariate structure. IQR's 40% agreement is a feature, not a bug: it cheaply surfaces univariate extremes that the ML methods sometimes miss, and the disagreement itself is informative.

3. **The features that drive flags are operationally meaningful.** The IQR analysis surfaces lube-oil temperature and fuel pressure as the dominant flag drivers — both of which an engineer would expect on physical grounds (lube-oil thermal margin is the classical early warning, fuel pressure swings track injector-pump health). When the model agrees with the domain, you trust it; when it disagrees, you investigate.

## What I'd do next in production

This notebook is the analysis layer. To turn it into a system, the missing pieces are:

- **Time context.** The current setup treats each cycle as independent. In production, flags should consider rolling windows (a single anomalous reading is noise; a five-cycle drift is a developing fault). A simple "n-of-last-m" overlay on top of these models would tighten precision substantially.
- **Per-vessel calibration.** A 2,000-RPM extreme on a high-speed ferry is normal; the same reading on a slow-steaming bulk carrier is a hard fault. Models should be fit per vessel class, not pooled.
- **Cost-aware thresholds.** A missed flag at sea costs more than a missed flag dockside. The "5% contamination" assumption should be replaced with an asymmetric cost function tied to vessel state.
- **Alert routing, not just detection.** The output of this layer is a shortlist; the production system needs to know which engineer is on duty, what other flags are open, and whether this anomaly correlates with recent maintenance.

## Getting started

```bash
# 1. Install dependencies (Python 3.10+)
pip install -r requirements.txt

# 2. The notebook downloads the dataset directly from the course's public
#    GitHub mirror, so no separate data step is needed. See data/README.md
#    for the URL and schema.

# 3. Open the notebook
jupyter lab notebooks/
```

## Why this dataset

Three reasons. First, it is **real industrial IoT data** with the awkwardness real sensor traces have — extreme outliers that are obviously bad readings rather than physically plausible faults, features on wildly different scales, no labels to train against. Second, the **physical interpretability** of the six channels means model output can be sanity-checked against engineering reasoning, which is the right rehearsal for any production deployment. Third, marine propulsion sits in the same broader family as the [aircraft engine RUL project](https://github.com/gurdeep-munde/cmapss-predictive-maintenance) — both are condition-monitoring problems on rotating machinery, and the contrast between supervised RUL prediction and unsupervised anomaly flagging is itself a useful framing for industrial-DS interviews.

## Author

Gurdeep Munde — Manufacturing Engineer and Cambridge Data Science Career Accelerator graduate.
[GitHub](https://github.com/gurdeep-munde)
