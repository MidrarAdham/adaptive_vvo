# House-15 BESS voltage-control experiment

Run from the repository root:

```bash
poetry run python gld/run_house15_bess.py
```

`house15.glm` is a separate copy of the current 15-house base case, with the
upstream swing source and line retained. The base case and voltage sweep are
unchanged. Its profiles and recorder paths are relative to this folder.

A child triplex meter at house 15 connects a GridLAB-D FOUR_QUADRANT inverter
and internal lithium-ion battery. Inverter rating is 5 kVA, P_Out is zero,
and Q_Out is limited externally to ±5,000 var. Positive Q injects reactive
power and was verified to increase house voltage. The battery is assumed to
have 10 kWh capacity, 400 V DC nominal voltage, 50% initial SOC, 5 kW rating,
and 90% round-trip efficiency. These are placeholders because only inverter
apparent-power rating was specified. SOC stays at 50% with zero real-power
transfer in this model; auxiliary/loss-driven battery depletion is not studied.

The Python controller first evaluates zero Q and both Q limits, then uses
per-minute bisection where the target is bracketed. It generates a tape-player
schedule for Q_Out targeting |V_1|/120 = 1.00. Balanced house loads make the two
legs equal in magnitude. Commands are checked against actual inverter VA_Out,
including zero P and the 5 kVA limit. Outside the reachable range the command
saturates at the appropriate limit. The target itself does not force voltage.

This is an offline steady-state controller with known demand, not a causal
feedback controller or dynamic inverter simulation. Only reactive dispatch is
used, so this is not a test of active battery charge/discharge. In the current
case the target is unreachable throughout the day: Q is +5 kvar at every
recorded minute. House-15 minimum improves from 0.90454 to 0.92078 pu; its
controlled maximum is 0.99051 pu. Fixing voltage to exactly 1 pu would require
additional capability or network changes.

Results are in `gld/results/house15_bess/`. `baseline/` retains matched Q=0
recordings; root CSVs are the final controlled run. `control_summary.csv`
reports target reachability, Q, and before/after house-15 voltages. All house
and primary/head recorder files remain available. Figures are
`gld/figures/house15_bess_control.svg` and `.png`, including upstream changes.
The supplied GLM runs with the last computed schedule; rerun the Python script
after changing loads or circuit parameters to recompute dispatch.
