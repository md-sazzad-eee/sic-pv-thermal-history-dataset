# Thermal-history dataset for a grid-connected SiC photovoltaic inverter

This repository contains the simulation data behind the manuscript "Thermal-History Ambiguity in Grid-Connected SiC Photovoltaic Inverters: Electrical Observability, Physics-Constrained Estimation and Uncertainty-Aware Derating".

The plant is a 100 kW, 400 V, 50 Hz grid-following inverter with an LCL filter and three Wolfspeed CAS300M12BM2 half-bridge modules on a forced-air heatsink. Electrical behaviour was simulated in MATLAB/Simulink and the module and heatsink were modelled in COMSOL Multiphysics 6.4.

## Contents

```
trajectories/                  one-hour operating trajectories with sensor noise (NumPy .npz)
fem/history_experiment/        finite-element runs: load histories ending at the same operating point
fem/endpoint_ablation/         finite-element runs: same history, different end points
fem/rom_validation/            finite-element runs used to validate the reduced-order model
fem/rom_validation_fine_step/  repeat of case rv5_aggr with a smaller finite-element time step
fem/solver_tolerance_check/    repeat of case dur60_H1 with a tighter solver tolerance
fem/calibration_and_steps/     calibration, mesh convergence, step responses, temperature field
rom/thermal_rom.json           six-pole Foster reduced-order thermal model
parameters/                    all plant parameters and their sources
SHA256SUMS.txt                 checksums of every file
```

## Trajectory datasets

Each file holds a set of independent trajectories of 3600 s sampled every 1 s (3601 samples, t = 0 to 3600 s). The plant was the averaged Simulink inverter model coupled to the reduced-order thermal model in `rom/`. The first 1200 s of every trajectory were used only as history and were not used as estimation targets in the paper.

| File | Trajectories | Condition |
|---|---|---|
| train_part1.npz, train_part2.npz | 300 + 300 | training domain: ambient 20 to 40 °C, cooling factor 0.9 to 1.1, 10 kHz, reactive power up to 0.2 pu, segments 20 to 600 s |
| val.npz | 120 | training domain (validation) |
| cal.npz | 120 | training domain (conformal calibration) |
| T1_nominal.npz | 150 | training domain (test) |
| T2_hot_ambient.npz | 100 | ambient 45 to 60 °C |
| T3_degraded_cooling.npz | 100 | cooling factor 0.6 |
| T4_fsw_8k.npz, T4_fsw_12k.npz, T4_fsw_16k.npz | 60 each | switching frequency 8, 12, 16 kHz |
| T5_aggressive.npz | 100 | segments 5 to 60 s, ramps 0.5 to 3 s |
| T6_long_dwell.npz | 100 | segments 600 to 1800 s |
| T7_unseen_PQ.npz | 100 | reactive power between 0.25 and 0.45 pu |

The training set is split into two files only to stay below the GitHub file size limit; concatenating them along the first axis gives the original 600 trajectories in order.

Arrays in every file (N trajectories, 3601 samples):

| Key | Shape | Content |
|---|---|---|
| F | N x 3601 x 11 | measured signals with sensor noise, columns listed below |
| Y | N x 3601 x 4 | true temperatures (°C): MOSFET junction, diode junction, case, heatsink |
| O | N x 3601 x 4 | estimate of the same four temperatures from the reduced-order model driven by F, assuming nominal cooling (°C) |
| P_true, Q_true | N x 3601 | noise-free active power (W) and reactive power (var) |
| PM, PD | N x 3601 | MOSFET and diode loss per switch position (W) |
| Ipk | N x 3601 | noise-free peak phase current (A) |
| hfac | N x 3601 | cooling factor (1 = nominal convection) |
| Ta_true | N x 3601 | true ambient temperature (°C) |
| meta | N x 7 | random seed, initial ambient (°C), cooling factor, switching frequency (Hz), on-state resistance scale, switching energy scale, completion flag |

Columns of F, in order: DC-link voltage (V), converter current d and q components (A, peak), active power (W), reactive power (var), grid voltage magnitude (V, peak phase), grid current magnitude (A, peak), modulation index, switching frequency (kHz), PLL frequency (rad/s), measured ambient temperature (°C).

Sensor model in F: current and voltage channels carry white noise of 0.5 % of rated value and a gain error of 0.3 % (one draw per trajectory); the ambient channel carries 0.2 K white noise and a bias of 0.5 K standard deviation (one draw per trajectory). Modulation index, switching frequency and PLL frequency are noise-free.

## Finite-element runs

Every case has an input file and an output file. Inputs (`*_input.csv`): time t (s), peak phase current Ipk (A), DC-link voltage Vdc (V), switching frequency fsw (Hz), ambient temperature Ta (°C), cooling factor hfac. Outputs (`*_output.csv`): time t (s), MOSFET junction temperature as die average TjM and maximum TjMmax, diode junction TjD, case Tcase, heatsink surface Ths (all °C), MOSFET and diode loss per switch PM_sw and PD_sw (W, computed inside the finite-element model from the die temperatures), Ipk (A) and Ta (°C).

History experiment case names: `main` (300 s segments, 35 °C ambient, nominal cooling, 10 kHz), `dur60` and `dur900` (60 s and 900 s segments), `h06` (cooling factor 0.6), `f16k` (16 kHz). H0 holds 0.6 pu throughout; H1 to H4 follow different load histories and then reach 0.6 pu at the same instant (t = 1240 s in the `main` cases).

Case `rv5_aggr` in `rom_validation` was solved with the production time-step settings; its repeat in `rom_validation_fine_step` uses a maximum step of 0.5 s and a relative tolerance of 1e-4 and should be preferred for ramps shorter than about 3 s.

## Reduced-order model

`rom/thermal_rom.json` contains the cooling-factor grid `hgrid`, the time constants `tau` (s) and the residues `R` (K/W) of the Foster network from the MOSFET and diode heat sources to the four outputs, together with the fit errors. Values between grid points are obtained by linear interpolation in 1/hfac.

## Loading

Python:

```python
import numpy as np
d = np.load("trajectories/T1_nominal.npz")
F, Y = d["F"], d["Y"]          # measured signals, true temperatures
```

MATLAB: the .npz files are zip archives of .npy arrays and can be read with Python from MATLAB, for example `py.numpy.load("trajectories/T1_nominal.npz")`.

## License

The data are released under the Creative Commons Attribution 4.0 International licence (CC BY 4.0).

## Citation

Please cite the associated article when using these data. Citation details will be added after publication.
