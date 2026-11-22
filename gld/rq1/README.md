# RQ1 studies

Every study holds Node4 (the transformer primary) at 1.00 pu as the swing node, runs a full day, and uses the
same inverter: 7.6 kVA, P = 0, constant Q all day, either +3.344 kvar (IEEE 1547, 44% of rating) or +7.6 kvar
(full rating). Shared code is in `common.py`.

Run everything from the repository root, for example `poetry run python gld/rq1/04_eight_houses/run.py`.
Each study writes `cases/` (generated glm files) and `results/` (recorder CSVs, `tidy.csv`, figures).

## Studies

| Folder | Model | What it does | Book |
|---|---|---|---|
| `04_eight_houses/` | **8-house baseline** | Base day; one inverter at each house in turn at 44% and full Q | ch4: all current figures and worked problems |
| `00_calibration/` | 15-house | Chooses the 40 ft main spacing (house 15 between 0.95 and 0.97 pu at 20:46) | ch5 calibration table |
| `01_base_day/` | 15-house | Base day, no inverters | Superseded by 04 |
| `02_one_inverter_h15/` | 15-house | One inverter at house 15, 44% Q | Superseded by 04 |
| `03_location_sweep/` | 15-house | One inverter at each house; full Q at houses 1, 8, 15; angle plot | Superseded by 04 |
| `hand_calcs/` | both | Backward/forward sweep hand calculations that reproduce GridLAB-D | ch4 worked problems (`ch4_numbers_8.py`) |
| `archive/` | 15-house | Earlier swing-voltage sweep approach and the location grid plot | ch5 operating-point table |

## Baseline study (04)

| Script | Output |
|---|---|
| `run.py` | Base day -> `results/tidy.csv`, `results/profile_map.csv` |
| `run_inverters.py` | 16 runs -> `results/q44/hNN/`, `results/full_q/hNN/`; checks Node4 = 1.00 pu and the inverter's P and Q every minute |
| `plot_base_day.py` | `base_day_8.svg`: demand and tap voltages |
| `plot_inverter_end.py` | `day_inverter_h08.svg`: one inverter at house 8 |
| `plot_profiles.py [--full-q]` | `day_location_profiles_8[_full_q].svg`: voltages without and with the inverter at houses 1, 4, 8 |

The book uses copies of these figures in `docs/figures_qmd/`; copy them again after re-plotting.

## `common.py`

| Name | Meaning |
|---|---|
| `MODEL`, `HOUSES` | Baseline: `gld/basecase/residential_8house.glm`, houses 1-8. Default for `run_day` |
| `MODEL_15`, `HOUSES_15` | 15-house model `gld/basecase/4_node.glm`, used by studies 00-03 |
| `RATING`, `Q44` | 7,600 VA and 3,344 var |
| `run_day(cases, results, name, inverters=(), houses=HOUSES, model=None)` | Builds the case (Node4 swing at 1.00 pu, recorders on every tap and house), runs GridLAB-D, writes `tidy.csv` |
| `add_inverters`, `head_voltages`, `style` | Inverter objects, Node4/secondary voltages, plot style |

`tidy.csv` columns: `timestamp, house, tap_v1_pu, tap_v2_pu, demand_kw, demand_kvar` (one row per house per
minute; `house` is the tap position).
