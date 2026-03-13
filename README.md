# HyDesign — Hybrid Power Plant Design & Optimisation

A Python framework for the design, control, and optimisation of utility-scale hybrid power plants combining wind, solar, battery storage, and Power-to-Hydrogen (P2X) electrolysers.

Based on the [DTU HyDesign framework](https://gitlab.windenergy.dtu.dk/TOPFARM/hydesign) (Murcia Leon et al., Wind Energy Science, 2024).

---

## What is in this repository

### Three standalone model notebooks

Each notebook explains one physical model from first principles — inputs, outputs, capabilities — and can be run independently without the full HyDesign optimisation stack.

| Notebook | Model | Key capabilities |
|---|---|---|
| `hydesign/notebooks/wind_model_test.ipynb` | Wind farm | Power curves, wake losses, degradation, directional dispatch |
| `hydesign/notebooks/battery_model_test.ipynb` | Battery storage | Rainflow cycle counting, stress degradation, SoH, replacement |
| `hydesign/notebooks/electrolyser_model_test.ipynb` | Electrolyser (P2X) | PEM/Alkaline efficiency, EMS optimisation, LCOH, degradation |

### Wind model notebook

Covers 6 capabilities:

1. **Single turbine power curves** — reads `genWT_v3.nc` (NetCDF surrogate) and interpolates to any specific power (W/m²)
2. **Wake-affected farm power curve** — reads `genWake_v3.nc` and computes wake losses as a function of Nwt and farm density
3. **Wind time series** — applies farm power curve to hourly wind speed data to get MW output
4. **Degradation** — models power curve shift over 25 years using two approaches (shift method and loss factor), controlled by a `share` parameter
5. **Directional power curves** — direction-dependent wake losses using 12 azimuth sectors
6. **PyWake integration** — optional high-fidelity wake model (requires PyWake)

### Battery model notebook

Covers 5 capabilities:

1. **Rainflow cycle counting** — decomposes irregular SoC time series into individual charge/discharge cycles
2. **Stress factors** — per-cycle damage as a function of DoD, mean SoC, and temperature (Xu et al., IEEE Trans. Smart Grid, 2016)
3. **Non-linear SoH model** — two-regime capacity loss with knee-point acceleration below SoH = 0.92
4. **Battery replacement** — tracks cumulative LoC across multiple battery replacements over 25-year lifetime
5. **Thermal de-rating** — piecewise linear capacity reduction at low temperatures (Lv et al., Energies, 2021)

### Electrolyser notebook

Covers 6 capabilities:

1. **Efficiency curves** — PEM vs Alkaline, both efficiency (%) and production (kg/h/MW) forms
2. **Greedy H2 dispatch** — send all available wind to the electrolyser, no optimisation
3. **EMS CPLEX optimisation** — MILP maximises electricity revenue + H2 revenue simultaneously with battery and H2 storage
4. **Cost model** — CAPEX/OPEX breakdown and LCOH sensitivity to electrolyser size
5. **Annual economics** — full-year optimised dispatch, revenue split, LCOH comparison vs greedy baseline
6. **Degradation** — 25-year H2 production decline from combined wind and PEM efficiency loss

---

## Installation

### Option 1 — conda environment (recommended)

```bash
conda create -n hydesign python=3.11
conda activate hydesign
pip install -r requirements.txt
pip install -e hydesign/
```

### Option 2 — pip only

```bash
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
pip install -e hydesign/
```

### CPLEX (required for EMS optimisation)

The electrolyser EMS notebook requires IBM CPLEX. The free Community Edition supports up to 1000 variables — sufficient for the notebooks (batch size 62 hours ≈ 870 variables).

Download from: https://www.ibm.com/products/ilog-cplex-optimization-studio (free academic/community edition available)

Install the Python API:
```bash
pip install cplex docplex
```

---

## Quick start

```bash
conda activate hydesign
jupyter notebook hydesign/notebooks/wind_model_test.ipynb
```

Run all cells top to bottom. Each notebook is self-contained.

---

## Repository structure

```
hydesign_simple/
├── README.md                        ← this file
├── requirements.txt                 ← Python dependencies
└── hydesign/                        ← HyDesign package
    ├── hydesign/
    │   ├── wind/                    ← wind model (power curves, wake, degradation)
    │   ├── battery_degradation.py   ← battery SoH model
    │   ├── ems/                     ← energy management system (CPLEX MILP)
    │   ├── costs/                   ← CAPEX/OPEX/LCOH models
    │   ├── look_up_tables/          ← genWT_v3.nc, genWake_v3.nc surrogates
    │   ├── examples/                ← weather CSVs, electrolyser efficiency curves
    │   └── tests/test_files/        ← reference outputs for regression tests
    └── notebooks/
        ├── wind_model_test.ipynb
        ├── battery_model_test.ipynb
        └── electrolyser_model_test.ipynb
```

---

## Key concepts

### NetCDF look-up tables

The wind model uses two pre-computed surrogate files instead of running a full aerodynamic simulation:

- `genWT_v3.nc` — single turbine power curve (`pc`) and thrust coefficient (`ct`) indexed by `(specific_power, wind_speed)`
- `genWake_v3.nc` — farm wake loss fraction (`wl`) indexed by `(specific_power, Nwt, wind_MW_per_km2, wind_speed)`

Both are read with `xarray` and interpolated to any design point in microseconds.

### Specific power

The key turbine design parameter used throughout: `specific_power = rated_power / rotor_area` (W/m²). Lower specific power (larger rotor for the same generator) gives higher capacity factor. Typical range: 150–360 W/m².

### EMS optimisation

The CPLEX MILP maximises `sum(electricity_price × P_grid + H2_price × m_H2) - penalty` subject to power balance, battery dynamics, H2 storage dynamics, and the electrolyser piecewise-linear production curve. The year is solved in 62-hour batches to stay within the CPLEX Community Edition variable limit.

---

## Citation

If you use HyDesign in your work:

> Murcia Leon, J. P., Habbou, H., Friis-Møller, M., Gupta, M., Zhu, R., and Sørensen, P. E.: *HyDesign: a tool for sizing optimization of grid-connected hybrid power plants including wind, solar photovoltaic, and lithium-ion batteries*, Wind Energ. Sci., 9, 1415–1451, 2024.

---

## License

MIT — see `hydesign/LICENSE`
