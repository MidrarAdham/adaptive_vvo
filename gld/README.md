# Residential secondary feeder

Run from this directory:

```bash
gridlabd 4_node.glm
poetry run python plot_voltages.py
```

The filename is retained for compatibility; this is no longer the IEEE four-node
feeder. It is a simplified North American-style residential secondary circuit:

```text
Fixed 2.4 kV source (Node4)
  |
50 kVA, 120/240 V service transformer
  |
mr_meter1 (residential feeder head)
  | 100 ft
 tap_1 ---- 30 ft service ---- house_1
  | 100 ft
 tap_2 ---- 30 ft service ---- house_2
  |
 ... successive 100 ft main segments ...
  |
 tap_30 --- 30 ft service ---- house_30
```

The main extends 3,000 ft from the transformer to the last tap. Each main segment
carries the combined current of all houses downstream. House profiles 1 through
30 are assigned individually. The active scenario uses 1.5x full-day demand
from `datasets/scaled_1p5/`; the original files in `datasets/` are unchanged.
To restore baseline demand, remove `scaled_1p5/` from the 30 player file paths. Profile values are VA; `test.py` converts original
kW values to W at unity power factor. Do not rescale the existing files again.

## Study assumptions

- The upstream network is replaced by an ideal 2.4 kV single-phase source.
  Its recorded voltage is fixed; use mr_meter1 to study the residential head.
- Transformer rating is an assumed 50 kVA, with the existing 0.01 pu resistance
  and 0.06 pu reactance retained. These are study inputs, not measured equipment data.
- Main and services retain the existing triplex conductor configuration:
  0.57 ohm/mile resistance and 0.0111 ft geometric mean radius.
  No thermal rating or utility construction standard has been validated.
- Each house is a constant-power 240 V equivalent (`constant_power_12`).
  This balances the two 120 V legs and preserves total measured demand. It does
  not model individual appliances, 120 V leg imbalance, or voltage-dependent loads.
  Both recorded legs will therefore have nearly equal magnitudes by design.
- Reactive demand is zero in the input profiles. Add measured reactive demand
  before using the model for power-factor-sensitive studies.

This is a plausible study topology, not a calibrated utility feeder. Match
conductor data, transformer impedance, service lengths and loading to a real
circuit before treating it as representative of a particular neighborhood.

## Outputs

- `results/voltage_a.csv`: residential feeder-head leg voltages.
- `results/house_N_voltage.csv`: each house's leg voltages after its service.
- `results/node4_voltage.csv`: fixed upstream source voltage.
- `results/service_power.csv`: feeder-head W and var.
- `results/voltages.png`: per-unit voltage plots (120 V house/head base,
  2,400 V upstream base). Edit settings at the top of `plot_voltages.py`.

The active model now contains 30 houses. Profiles 11–30 were imported from
`/home/midrar/Desktop/IEEE_4_Node_Feeder_Exercise/gld_project_datasets/`:
source `house power (kW)` values were multiplied by 1,000 for baseline VA files
(unity PF), and by another 1.5 for the active scaled copies. Original source
files were left unchanged.

**The 30-house simulation does not complete.** With the retained 50 kVA
transformer, existing conductors, 100 ft tap spacing, and 1.5x demand, GridLAB-D
reports line continuous/emergency rating violations and stops with a convergence
failure near 04:45 local time. Current result CSVs and the regenerated plot cover
only the partial run, not a full day. These results must not be treated as a
validated operating case. A realistic expansion needs a redesigned main or
multiple transformer-fed secondary circuits; transformer enlargement alone
does not address the long shared secondary main.

GridLAB-D property reference:
https://gridlab-d.readthedocs.io/en/docs/docs/3.0%20-%20Modeling%20Reference/Modules/Powerflow/Classes/21-triplex_load/

## Upstream line addition

`upstream_source` is now the fixed 2,400 V swing reference. It feeds the
variable-voltage transformer primary `Node4` through `upstream_line`.
The upstream line is 1,000 ft and uses the original `LC300` configuration,
`LS200` spacing, phase conductor `OH100` (0.306 ohm/mile, 0.0244 ft GMR),
and neutral `OH101` (0.592 ohm/mile, 0.00814 ft GMR). For this AN line,
the A-to-neutral spacing is 5.656854 ft; B/C geometry is retained for reference.
GridLAB-D computes impedance from these conductor and geometry inputs.
The line length remains a study assumption.
`results/node4_voltage.csv` now records a variable primary voltage.
No inverter or reactive-power controller has been added in this step.
The current enabled-house configuration completed a full-day test after this
addition; earlier 30-house failure notes describe the previous configuration.

## Node4 voltage sweep

Run `poetry run python gld/sweep_node4.py` from the project root. Edit the
settings at the top of the script; no command-line arguments are required.
The sweep runs the currently enabled houses for the base-case clock window
at 0.95 through 1.05 pu in 0.01 pu steps (2,400 V primary base).

Each generated case removes upstream_source and upstream_line and makes Node4
the swing reference. This imposes the exact Node4 boundary voltage rather than
merely changing its initial guess. The base-case GLM is never modified; this
experiment does not represent inverter control or retain upstream-line losses.
Loads, transformer, service lines, and enabled-house selection are preserved.

Each scenario's GLM, run.log, and time-series CSVs are in
`gld/results/node4_sweep/node4_X.XXpu/`. The combined `summary.csv` reports
per-house minima/maxima across both legs, minimum timestamps, and sample counts.
The figure is `gld/figures/node4_sweep.png`. Daily minima at different houses
may occur at different timestamps; they are not necessarily a simultaneous
voltage profile. The current 15-house run completed all 11 cases.
