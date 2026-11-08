# All-house reactive dispatch toward 0.95 pu

Run from the project root:

```bash
poetry run python gld/run_all_house_inverters.py
```

The script generates `all_houses.glm` from `gld/basecase/4_node.glm`, retaining
its source, upstream line, transformer, active houses, distinct scaled demand
profiles and clock. It adds one inverter/BESS at each of the 15 houses using a
child triplex meter at the house bus. Earlier models/studies are unchanged.

## Assumptions and method

- All fifteen inverters operate simultaneously at zero real power.
- Each inverter is 5 kVA and constrained to -5 through +5 kvar. Positive Q
  injects; negative Q absorbs. There is no active battery dispatch.
- Each house has target 0.95 pu on the 120 V leg base. Loads are balanced 240 V
  equivalents, and both recorded legs have equal magnitudes in this case.
- The upstream source remains 2,400 V SWING. Node4, the transformer secondary,
  taps and house voltages respond to the combined reactive dispatch.
- Battery assumptions match the earlier case: 10 kWh, 400 V DC, 50% initial
  SOC, 90% round-trip efficiency. Energy capacity is not exercised at zero P.
- This is coordinated offline dispatch, not automatic local Volt–VAR control.
  Minute-specific voltage sensitivities are estimated by fifteen 0.25 kvar
  perturbation runs. Bounded least squares minimizes the sum of squared voltage
  errors across all houses, with a small Q regularizer. Repeated full-day power
  flows verify nonlinear responses; Q updates are limited to 1 kvar per iteration.
  The best combined squared-error solution is retained separately at each minute.
- The objective is aggregate tracking error; it does not guarantee improvement
  at each individual house. Network equipment limits are not optimization
  constraints, though this run had no overload warnings and its peak secondary
  apparent power is reported. The result is a local numerical solution and does
  not prove global feasibility/infeasibility of exact simultaneous regulation.

## Current results

The run covers 1,439 minute samples on 2025-09-01, with 21,585 house-minute
observations. All actual inverter outputs were checked for zero P, matching
commanded Q, and apparent power no greater than 5 kVA. Both legs were checked
for equal magnitude. Combined tracking error did not increase at any timestamp.

- RMS error from 0.95 pu: 0.02304 before, 0.00873 after control.
- 61.6% of house-minute observations lie within 0.945–0.955 pu.
- At no recorded minute are all fifteen houses within that band together.
- House 1 remains at 0.96405–0.98609 pu, despite -5 kvar throughout the day.
- House 15 is at 0.93551–0.95077 pu, versus 0.90454–0.97546 pu with zero Q.
- At 20:46 house 15 rises from 0.90454 to 0.93551 pu.
- Peak transformer-secondary loading is 30.03 kVA on the 50 kVA unit.

Thus the computed bounded dispatch improves overall tracking substantially,
but does not establish close simultaneous tracking at every house. The effect
is strongly coupled: the inverters absorb near the head and inject farther
downstream, with several reaching their capability limits.

## Files

`gld/results/all_house_inverters/` contains:

- `house_summary.csv`: individual voltage ranges, errors, Q ranges and tracking fractions.
- `voltages_and_q.csv`: per-minute, per-house baseline/control voltage and actual Q.
- `report.json`: aggregate metrics and assumptions.
- `baseline/`: raw recorder CSVs with all fifteen inverter Q setpoints at zero.
- Root recorder CSVs: final controlled run, including all inverter actual P/Q.
- `q_house_N.csv`: final Q schedule for each inverter.
- `sensitivity.npz`: minute-specific dV/dQ matrices and timestamps.
- `iterations.csv` and logs: error convergence and simulation diagnostics.

`gld/figures/all_house_inverters/all_houses_target_0p95.svg` and `.png` show
controlled voltages, Q dispatch, and the simultaneous 20:46 voltage profile.
A documentation copy is `docs/figures_qmd/all_houses_target_0p95.svg`.

Running the generated GLM directly follows the saved schedules. To recalculate
Q after changing demand or circuit data, update the base case and rerun the
Python script. Configure TARGET, ITERATIONS and TOLERANCE_PU at its top.
