# Isfahan Shelter Location Optimization

A mixed-integer programming model that determines the minimum number of emergency shelter units required across Isfahan's 15 municipal districts, and how evacuees should be allocated between district-level shelters and subway-station evacuation platforms during a disaster (e.g., earthquake) scenario.

The model is built with [GAMSPy](https://gamspy.readthedocs.io/) and formulated as a facility location / demand allocation problem, using real district population, geographic boundaries, and risk-target data for Isfahan.

## Problem Overview

Given:
- 15 city districts, each with a known population and a set of identified risk "targets" (e.g., high-risk structures or hazard points)
- 20 subway stations that can double as evacuation platforms with fixed capacity
- Distances between districts and between districts and subway stations (Haversine distance)

The model decides:
- How many shelter units to build in each district
- How evacuees from each district are split between reachable district shelters and reachable subway stations

...subject to:
- **Demand coverage** — every district's population must be fully allocated to a shelter or subway station
- **Reachability** — evacuees can only be routed to a shelter/station within an acceptable evacuation radius (default: 4.5 km)
- **Capacity** — each shelter unit and each subway platform has a fixed evacuee capacity
- **Risk-based minimum coverage** — districts with a higher density of risk targets are required to have a baseline number of shelter units

The objective is to **minimize the total number of shelter units built** across all districts.

## Repository Structure

```
.
├── Data/                       # Input datasets (population, district geometry, risk targets)
├── latex_MathematicalModel/    # LaTeX export of the formal mathematical model
├── Model.ipynb                 # Main notebook: data processing, model formulation, and solve
├── LICENSE                     # MIT License
└── README.md
```

### Data

- **District population** — population figures for Isfahan's 15 districts (Excel).
- **District geometries** — GeoJSON boundaries for Isfahan's districts, used to compute district centroids.
- **Refined risk targets** — point-level risk/hazard data used to determine each district's baseline shelter requirement.

### Model

`Model.ipynb` walks through the full pipeline:

1. **Load data & geometries** — reads population figures and district polygons, computes centroids, and counts risk targets per district.
2. **Distance & reachability matrices** — computes Haversine distances between district centroids and from districts to subway stations, then derives binary reachability matrices using a configurable evacuation radius.
3. **Mathematical model** — formulates the problem as a Mixed-Integer Program (MIP) in GAMSPy:
   - **Sets**: districts (`i`), subway stations (`k`)
   - **Decision variables**: `Y[i]` (integer number of shelter units per district), `x[i, i']` and `w[i, k]` (continuous evacuee flows to shelters and subway stations)
   - **Constraints**: demand coverage, reachability, shelter/subway capacity, and minimum risk-based coverage
   - **Objective**: minimize total shelter units
4. **Solve & extract results** — solves the MIP and reports the number of shelter units required per district.
5. **LaTeX export** — exports the solved model's formulation to `latex_MathematicalModel/` for documentation/reporting.

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

## Notes

- The evacuation radius (`d_max_km`), shelter unit capacity, and subway platform capacity are configurable parameters at the top of the relevant notebook cells and can be adjusted to reflect different planning assumptions.
- Subway station coordinates are currently hardcoded in the notebook based on known station locations along Isfahan's metro line.

## License

This project is licensed under the [MIT License](LICENSE).

## Author

**Shayan** — Industrial Engineering student, University of Tehran
