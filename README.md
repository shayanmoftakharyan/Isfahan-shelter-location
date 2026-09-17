# Isfahan Shelter Location Optimization

A mixed-integer programming model that determines the minimum number of emergency shelter units required across Isfahan's 15 municipal districts, and how evacuees should be allocated between district-level shelters and subway-station evacuation platforms during a crisis scenario (e.g., armed conflict or an attack on strategic infrastructure).

The model is built with [GAMSPy](https://gamspy.readthedocs.io/) and formulated as a facility location / demand allocation problem, using real district population, geographic boundaries, and strategic-target location data for Isfahan.

## Problem Overview

Given:
- 15 city districts, each with a known population
- A set of strategic targets in and around the city (military bases, defense complexes, industrial parks, energy facilities) that carry elevated attack risk
- 20 subway stations that can double as evacuation platforms with fixed capacity
- Distances between districts, between districts and strategic targets, and between districts and subway stations (Haversine distance)

The model decides:
- How many shelter units to build in each district
- How evacuees from each district are split between reachable district shelters and reachable subway stations

...subject to:
- **Demand coverage** — every district's population must be fully allocated to a shelter or subway station
- **Reachability** — evacuees can only be routed to a shelter/station within an acceptable evacuation radius (default: 4.5 km)
- **Capacity** — each shelter unit and each subway platform has a fixed evacuee capacity
- **Risk-based minimum coverage** — districts closer to strategic targets are required to guarantee shelter capacity for a minimum share of their population, scaled by a distance-decayed exposure score (closer targets contribute more risk than distant ones)

The objective is to **minimize the total number of shelter units built** across all districts.

## Repository Structure

```
.
├── Data/                       # Input datasets (population, district geometry, strategic targets)
├── latex_MathematicalModel/    # LaTeX export of the formal mathematical model
├── Model.ipynb                 # Main notebook: data processing, model formulation, and solve
├── LICENSE                     # MIT License
└── README.md
```

### Data

- **District population** — population figures for Isfahan's 15 districts (Excel).
- **District geometries** — GeoJSON boundaries for Isfahan's districts, used to compute district centroids.
- **Strategic targets** (`refined_targets.npy`) — point locations of military bases, defense/Basij complexes, checkpoints, heavy industrial parks, and energy facilities in and around Isfahan. Used to compute each district's strategic-risk exposure score.

### Model

`Model.ipynb` walks through the full pipeline:

1. **Load data & geometries** — reads population figures and district polygons, computes centroids, and loads strategic target locations.
2. **Distance & reachability matrices** — computes Haversine distances between district centroids and from districts to subway stations, then derives binary reachability matrices using a configurable evacuation radius (`d_max_km`).
3. **Strategic risk exposure** — computes a continuous, distance-decayed risk score per district: every strategic target contributes `1 / (1 + distance)` to each district's score, so nearby targets contribute more risk than distant ones and no target is excluded outright. Scores are normalized to `[0, 1]` across districts (`risk_normalized`).
4. **Mathematical model** — formulates the problem as a Mixed-Integer Program (MIP) in GAMSPy:
   - **Sets**: districts (`i`), subway stations (`k`)
   - **Parameters**: `P[i]` (population), `Risk[i]` (normalized strategic-risk exposure), `A[i, i']` / `B[i, k]` (reachability), `CapSub[k]` (subway capacity)
   - **Decision variables**: `Y[i]` (integer number of shelter units per district), `x[i, i']` and `w[i, k]` (continuous evacuee flows to shelters and subway stations)
   - **Constraints**:
     - Demand coverage: `sum(x[i,·]) + sum(w[i,·]) == P[i]`
     - Reachability: flows only allowed within the evacuation radius
     - Shelter/subway capacity: flows into a facility can't exceed its capacity
     - **Risk-based minimum coverage**: `Y[i] * C_val >= risk_coverage_fraction * Risk[i] * P[i]` — districts with higher strategic-risk exposure must guarantee shelter capacity for at least `risk_coverage_fraction` (default 25%) of their population, scaled by how exposed they are relative to the most at-risk district
   - **Objective**: minimize total shelter units
5. **Solve & extract results** — solves the MIP and reports the number of shelter units required per district.
6. **LaTeX export** — exports the solved model's formulation to `latex_MathematicalModel/` for documentation/reporting.

## Requirements

- Python 3.9+
- [GAMSPy](https://gamspy.readthedocs.io/) (requires a GAMS installation/license — a free community license supports small models)
- pandas
- numpy
- shapely
- openpyxl (for reading `.xlsx` data)

Install dependencies:

```bash
pip install gamspy pandas numpy shapely openpyxl
```

## Usage

1. Clone the repository:
   ```bash
   git clone https://github.com/shayanmoftakharyan/Isfahan-shelter-location.git
   cd Isfahan-shelter-location
   ```
2. Ensure the `Data/` folder is present with the required population, geometry, and target files.
3. Open and run `Model.ipynb` end to end (Jupyter or JupyterLab):
   ```bash
   jupyter notebook Model.ipynb
   ```
4. Review the printed results table for the number of shelter units allocated per district, and check `latex_MathematicalModel/` for the exported formulation.

## Sample Results

A representative solve (`C_val = 20,000` per shelter unit, `CapSub_val = 15,000` per subway platform, `risk_coverage_fraction = 0.25`) reaches optimality in well under a second and requires **92 total shelter units** across the 15 districts, ranging from 1 unit (D3) to 10 units (D14).

## Notes

- The evacuation radius (`d_max_km`), shelter unit capacity, subway platform capacity, and `risk_coverage_fraction` are configurable parameters near the top of the relevant notebook cells and can be adjusted to reflect different planning assumptions.
- The strategic-risk exposure score currently treats every target equally (no severity/type weighting) and is not capped by the evacuation radius — all targets contribute to all districts' scores, just with distance decay. Both are intentional simplifications; a natural extension is weighting targets by type (e.g., an air base vs. a checkpoint) if such data becomes available.
- Subway station coordinates are currently hardcoded in the notebook based on known station locations along Isfahan's metro line.

## License

This project is licensed under the [MIT License](LICENSE).

## Author

**Shayan** — Industrial Engineering student, University of Tehran
