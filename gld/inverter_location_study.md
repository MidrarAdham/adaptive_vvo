# Inverter location, head voltage, reactive power and loading

Run from the repository root:

```bash
poetry run python gld/sweep_inverter_locations.py
```

Edit settings at the top of the script; no argparse is used. This study reads
`gld/basecase/4_node.glm` and creates independent cases. Earlier experiments and
base/BESS models are unchanged. The enabled houses are currently 1–15.
Exactly one 5 kVA inverter and one idle battery are installed in each generated
model, connected through a child meter at the selected house. No other house
has an inverter in that case. Locations are submitted from house 15 to house 1;
four independent simulations may run concurrently, without electrical coupling
between cases.

## Experimental design

- Inverter location: each of houses 15 through 1, separately.
- Node4 primary boundary: 0.95–1.05 pu in 0.01 pu increments; base 2,400 V.
- Q: -5 to +5 kvar in 1 kvar increments. Positive injects, negative absorbs.
- Inverter P: zero; its entire 5 kVA capability is available for reactive power.
- Loads: frozen at 2025-09-01 20:46 PDT, then multiplied by 0.75, 1.00, or 1.25.
  These factors apply to the current player files, which already use 1.5x
  scaling relative to the original datasets. Total demand is respectively
  16.215, 21.620 and 27.025 kW at the selected timestamp. Existing Q demand is zero.
- Each generated case replaces the upstream source/line with a swing at Node4
  to impose the specified voltage exactly. Thus Node4 cannot respond to Q in
  this experiment; the transformer secondary and all houses can respond.
- Transformer, main, service lines and per-house demand differences are retained.
- Battery assumptions follow the earlier case (10 kWh, 400 V, 50% SOC); P=0
  means energy capacity is not exercised here. This is a steady-state
  sensitivity study, not automatic voltage control or a battery dispatch study.

The full factorial design contains 15 × 11 × 11 × 3 = 5,445 operating points.
Every case records both legs of all 15 houses, transformer-secondary voltage,
primary voltage, inverter actual P/Q, and secondary power flow.

## Results

All 5,445 cases completed successfully. Validation checked actual Q dispatch,
5 kVA capability, exact Node4 boundary voltage, 81,675 house observations, and
identical Q=0 voltages across inverter locations at matched loading/head voltage.
Convergence alone is not an equipment loading or voltage-compliance assessment.

At Node4 = 1.00 pu and current loading, the house-15 voltage is 0.90498 pu with
Q=0. With +5 kvar injected at house 1, 8, or 15, it becomes respectively
0.91229, 0.91672, or 0.92102 pu. This demonstrates the importance of the shared
impedance between inverter and measurement locations. These values differ
slightly from the earlier Q sweep, which retained the upstream line instead
of fixing Node4.

Files under `gld/results/inverter_location_sweep/`:

- `summary.csv`: one row per operating point; own-house voltage, all-house
  minimum/maximum, secondary voltage, actual dispatch, load, and own-house/
  secondary voltage change relative to the matching Q=0 case.
- `house_voltages.csv`: one row per measured house per operating point, with
  both leg voltages and their changes from matched Q=0.
- `sensitivities.csv`: location-to-location central-difference dV/dQ near zero,
  calculated as (V at +1 kvar minus V at -1 kvar) / 2 kvar, at Node4 = 1 pu.
- `frozen_loads.csv`: individual house demand before the three loading factors.
- `manifest.json` and `basecase_template.glm`: settings and source-model snapshot.
- `house_NN/`: per-location result tables, final case model, final raw recorder
  files and log. Failed cases are preserved separately if any occur.

Raw files are reused within each location to avoid retaining thousands of
redundant recorder files. Enable KEEP_ALL_CASE_FILES to retain every model/log/
recorder set. The script regenerates the combined tables when rerun; it does
not resume an interrupted study automatically.

Figures under `gld/figures/inverter_location_sweep/`, in SVG and PNG:

- `location_q_response`: own-house voltage vs Q at each inverter location,
  showing 0.95, 1.00 and 1.05 pu head voltage at current loading.
- `voltage_sensitivity`: measured-house vs inverter-house sensitivity matrices
  at all three loading levels; these show upstream and downstream responses.
- `loading_and_location`: the lowest network voltage vs inverter location,
  comparing -5, 0 and +5 kvar at each loading level.

All 11 head-voltage values are retained in the CSVs. Results describe this
one frozen demand pattern; additional timestamps would test different
spatial demand distributions as well as total loading.

## Combined 3D figure

Run `poetry run python gld/figures/plot_location_sweep_3d.py` to plot existing
results without running simulations. `combined_3d.svg` and `.png` are saved
in `gld/figures/inverter_location_sweep/`. Columns show the three loading levels;
the top row shows voltage at the house hosting the inverter, and the bottom
row shows the lowest voltage among all houses. Axes are inverter location, Q,
and voltage; color distinguishes all 11 imposed Node4 voltage settings.
A documentation copy is in `docs/figures_qmd/inverter_location_combined_3d.svg`.
Set SHOW_PLOT=True to rotate the figure in a Matplotlib GUI. The connecting
meshes visualize discrete operating points; house locations are not continuous.
