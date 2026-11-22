# Radar Elbow Extension Simulation

A Python simulation of a passive coded radar sleeve that estimates how far a bowler’s elbow straightens during a cricket delivery. Eight markers on the upper arm and eight on the forearm produce distinct radar echoes. The simulator combines these echoes with synthetic marker positions to reconstruct elbow motion from the start of the delivery window to ball release.

## Example result

In the default delivery, the elbow starts at **30°** of flexion, reaches **33°** and releases at **8°**. The known extension is therefore **33° − 8° = 25°**.

| Measurement | Result |
|---|---:|
| Known elbow extension | 25.00° |
| Estimate from integrated radar velocity measurements | 24.64° |
| Estimate after combining radar measurements with marker positions | **24.89°** |
| Absolute error of the combined estimate | **0.11°** |
| Accepted frames | 81 of 81 |

![Simulated and reconstructed elbow flexion over an 80 ms delivery window, with the estimation error shown below](docs/figures/delivery.png)

The upper panel shows the reconstructed angle alongside the known motion. The lower panel shows their difference at each frame. These results describe one reproducible synthetic delivery. [View the reference output](examples/reference_summary.json)

## How it works

1. **Generate the motion.** The model moves the upper arm and forearm through a prescribed delivery, including elbow flexion, upper-arm rotation and forearm pronation.
2. **Simulate the radar echoes.** A frequency-modulated continuous-wave (FMCW) radar observes the sleeve directly and through two planar reflectors. Each marker has a distinct frequency-notch pattern. The simulation includes Doppler shifts, receiver noise and missed echoes.
3. **Reconstruct elbow motion.** Phase changes between successive radar chirps provide radial speed measurements. A joint motion model combines these measurements with noisy marker positions to estimate the elbow flexion rate and angle.
4. **Calculate extension.** The estimator measures the decrease from peak flexion to release. If the elbow passes through straight, the additional hyperextension is discounted. When too few observations support an estimate, the program reports `insufficient_geometry`.

A separate experiment tests **blind decoding**. Two coded markers occupy the same radar range cell and move in opposite directions. The decoder uses the mixed signal and the codebook to recover their identities and speeds in the presence of a nearby uncoded echo.

## Quick start

Use **Python 3.12**. NumPy is the only runtime dependency. From the repository root on macOS or Linux, run:

```bash
python3.12 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python src/simulate.py
```

The simulation writes two files:

| File | Contents |
|---|---|
| `results/summary.json` | Known and estimated extension, accepted frame count, reconstruction status and blind decoding results |
| `results/frames.csv` | Time, known flexion, echo count, estimated flexion rate, standard error and reconstructed angles for each frame |

## Try a different delivery

Copy the example configuration:

```bash
cp config/demo.json config/my_delivery.json
```

In `config/my_delivery.json`, `flexion_deg` gives the **initial, peak and release angles** in degrees. For example, `[6, 6, -12]` models an elbow that starts at 6° of flexion and ends 12° beyond straight. Its expected extension is **6°**.

Run the modified configuration and save its results separately:

```bash
python src/simulate.py --config config/my_delivery.json --out results/my_delivery
```

To repeat the default motion with a different realization of measurement noise and echo dropout, change the random seed:

```bash
python src/simulate.py --seed 43 --out results/seed_43
```

## Tests

```bash
python -m unittest discover -s tests -v
```

The six regression tests cover rigid marker alignment, overlapping-echo decoding, extension and hyperextension, insufficient measurement geometry, complete echo loss and reproducibility of the command-line seed override.

## Model assumptions

The delivery reconstruction uses synthetic labelled marker positions, calibrated reflector geometry and supplied start and release times. The default configuration uses 80° marker visibility and enlarged reflectors. Accuracy with a physical sleeve and radar remains unmeasured. [Read the measurement model](docs/MODEL.md)

## Contributing

To report a result, include the configuration, random seed, Python and NumPy versions, and relevant fields from `results/summary.json`. For a code change, run the regression tests and describe its effect on the default delivery or the specific case it addresses.
