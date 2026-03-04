# Thermal Fault Detector

A signal analysis tool that detects and visualizes electrical faults in power system recordings using thermal color heatmaps. It processes time-series current (I1) and voltage (V1) data, classifies each timestep into a 6-level thermal severity scale, and produces a **NVA (Normalized Vibrational Activity) score** and a **4-bit binary fault summary**.

---

## Features

- **Thermal color heatmaps** for current, voltage, and a fused channel
- **6-level severity scale**: Violet → Blue → Green → Yellow → Orange → Red
- **Fault-gradient coloring** for current: smooth Yellow→Orange→Red within the fault zone
- **ROC-only voltage coloring**: marks transition instants only — stable periods (whether at nominal voltage or 0 V) show Violet/Blue
- **State machine fault tracking** with configurable cooldown to prevent premature fault-exit
- **NVA score** (0.0 = perfect, 1.0 = total fault) with five named health states
- **Binary output** `[COLD | NORMAL | WARNING | DANGER]` with human-readable interpretation for all 16 combinations
- **Auto-detect file loader**: handles `.xlsx`, `.xls`, CSV (comma/semicolon), and TSV

---

## Input Format

The tool expects a spreadsheet or CSV file with **at minimum these three columns**:

| Column   | Description              |
|----------|--------------------------|
| `simOut` | Time axis (seconds)      |
| `V1`     | Voltage signal (Volts)   |
| `I1`     | Current signal (Amperes) |

Extra columns are ignored. Set `FILE_PATH` in **Cell 2** to point to your data file.

---

## Outputs

| Output | Description |
|---|---|
| Current heatmap | Fault-gradient coloring with ROC spikes |
| Voltage heatmap | ROC-only coloring, marks onset/offset transitions |
| Fused heatmap | `max(current_color, voltage_color)` per timestep |
| Color % breakdown | Time spent at each severity level |
| NVA score | Weighted scalar health index |
| Binary string | 4-bit dominant-state summary with interpretation |

---

## Color Scale

| Index | Color  | Meaning |
|-------|--------|---------|
| 0     | Violet | Cold / at rest |
| 1     | Blue   | Very low activity |
| 2     | Green  | Moderate activity |
| 3     | Yellow | Elevated / fault entry zone |
| 4     | Orange | Active fault — lower severity |
| 5     | Red    | Active fault — peak severity |

---

## NVA Score Scale

```
0.00 – 0.20  →  GOOD
0.20 – 0.40  →  OK
0.40 – 0.60  →  AVERAGE
0.60 – 0.80  →  FAULTY
0.80 – 1.00  →  SEVERE FAULT
```

---

## Key Parameters

Tunable constants defined at the top of **Cell 1**:

| Parameter | Default | Description |
|---|---|---|
| `REST_VALUE` | `120.0` | Idle signal level for both I1 and V1 |
| `REST_TOL` | `0.5` | Tolerance for rest detection (±V/A) |
| `FAULT_COOLDOWN` | `50` | Steps below 2× normal required to exit fault state |
| `HEIGHT` | `30` | Pixel height of heatmap bar |

Thresholds computed automatically from the data:

| Threshold | Description |
|---|---|
| `normal_I` | Median of the lower 50th percentile of active (non-rest) current samples |
| `thr_5x` | Fault entry threshold — `normal_I × 5` |
| `thr_2x` | Fault exit candidate — `normal_I × 2` |
| `fault_ceiling` | 99.9th percentile of absolute current (prevents single-spike distortion) |

---

## How It Works

### Current Coloring
Two zones are handled separately:

- **Normal zone**: color is the higher of ROC-based color and level-based color (Blue/Green/Yellow depending on how many multiples of `normal_I` the current is)
- **Fault zone** (current > `thr_5x`): color is the higher of ROC spike color and a fault-gradient color. The fault-gradient maps the current's position between `thr_5x` and `fault_ceiling` linearly to Yellow→Orange→Red, producing a natural severity gradient instead of locking the entire fault region to solid Red

A state machine governs fault entry and exit. Once entered, the fault state persists until current stays below `thr_2x` for `FAULT_COOLDOWN` consecutive steps.

### Voltage Coloring
Voltage uses ROC-only coloring. Rapid transitions (fault onset, recovery) produce Orange/Red spikes; stable periods at any voltage level show Violet/Blue. This prevents a sustained voltage drop from flooding the heatmap with red.

### Fused Heatmap
At each timestep, the fused color is `max(current_color, voltage_color)`. Current drives the sustained fault color; voltage contributes only at transition moments.

---

## Usage

1. Open the notebook in **Jupyter** or **Google Colab**.
2. Set `FILE_PATH` in Cell 2 to your data file path.
3. Run all cells in order (Cells 1 → 11).

```bash
jupyter notebook ThermalFaultDetector.ipynb
```

---

## Requirements

See `requirements.txt`. Install with:

```bash
pip install -r requirements.txt
```

---

## License

MIT
