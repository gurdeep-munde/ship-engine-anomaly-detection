# Dataset — Marine Engine Sensor Readings

A real condition-monitoring dataset published openly by the Cambridge Data Science programme. 19,535 cycles, six numeric channels, no missing values, no duplicates, no labels.

## How to obtain

The notebook reads the file directly from the course's public GitHub mirror — no manual download step is needed:

```python
import pandas as pd

url = "https://raw.githubusercontent.com/fourthrevlxd/cam_dsb/main/engine.csv"
df = pd.read_csv(url)
```

If you would rather have a local copy:

```bash
# From the project root (04-ship-engine-anomaly-detection/)
curl -sSL -o data/raw/engine.csv \
  "https://raw.githubusercontent.com/fourthrevlxd/cam_dsb/main/engine.csv"
```

## Schema

| Column | Type | Units | Notes |
|---|---|---|---|
| `Engine rpm` | int | revolutions/min | Idle ≈ 600, cruise band 600–900; observed max 2,239 |
| `Lub oil pressure` | float | bar | Mean ≈ 3.3; bimodal distribution suggests two operating loads |
| `Fuel pressure` | float | bar | Right-skewed; high tails track injector / pump health |
| `Coolant pressure` | float | bar | Right-skewed; low values suggest leak, high suggests blockage |
| `lub oil temp` | float | °C | Tight band 71–82; right tail indicates lubricant thermal stress |
| `Coolant temp` | float | °C | Mean ≈ 78; two extreme outliers (120 °C and 195 °C) likely real overheating events or sensor faults |

No timestamps. Rows are not labelled as "good" or "bad" — this is unsupervised territory.

## Provenance

Originally compiled by Devabrat (2022). Published under the Cambridge Data Science Career Accelerator (Course 1, Week 5 mini-project) for educational use.
