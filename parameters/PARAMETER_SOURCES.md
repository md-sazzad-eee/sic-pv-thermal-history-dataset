# Parameter sources

Tags: **DATASHEET** = Wolfspeed CAS300M12BM2 data sheet (rev. 2024, assets.wolfspeed.com);
**RULE** = standard design rule (formula given); **ASSUMPTION** = engineering choice for this study;
**CALIBRATED** = tuned so the FEM reproduces a datasheet value.

## Device (DATASHEET, CAS300M12BM2, per switch position)
| Quantity | Value | Condition |
|---|---|---|
| R_DS(on) | 4.17 mΩ (25 °C), 7.2 mΩ (150 °C) | V_GS = 20 V, I_D = 300 A; linear interpolation in T (ASSUMPTION: datasheet curve is mildly convex) |
| E_on | 3.4 / 3.0 / 2.9 mJ (25/125/150 °C) | V_DD = 600 V, I_D = 300 A, R_G(ext) = 0 Ω, L = 22.2 µH |
| E_off | 3.4 / 3.4 / 3.5 mJ | same |
| E_rr (SBD) | 2.04 / 2.17 / 2.18 mJ | same |
| R_th,jc MOSFET / SBD | 0.075 / 0.076 K/W (typ) | |
| T_vj,op max | 150 °C | |

Loss model (RULE): conduction per switch with synchronous rectification P_cond = R_DS(on)(T_j)·I_pk²/4;
switching P_sw = f_sw·(E_on+E_off)(T_j)·(V_dc/600)·(I_pk/(π·300)); SBD P_rr = f_sw·E_rr(T_j)·(V_dc/600)·(I_pk/(π·300))
(linear V and I scaling of datasheet energies; dead-time conduction neglected). Verified against event-based
loss accounting in the switching-function model.
Energies interpolated piecewise-linearly in T with linear extrapolation (identical in MATLAB, COMSOL and Python).

## Converter / grid
| Quantity | Value | Tag |
|---|---|---|
| Rating | 100 kW, 400 V L-L, 50 Hz | ASSUMPTION |
| V_dc | 750 V | ASSUMPTION (SVPWM, m ≈ 0.88 at rated) |
| C_dc | 3 mF | ASSUMPTION |
| L1 | 300 µH | RULE: L1 = V_dc/(6·f_sw·ΔI_max), ΔI_max = 20 % I_rated,pk (Reznik et al., IEEE TIA 2014) |
| C_f | 50 µF (Y) | RULE: ≤ 5 % of base capacitance (C_b = 1.99 mF) |
| L2 | 100 µH | RULE: resonance 2.6 kHz ∈ (10 f_g, f_sw/2) |
| R_d | 0.40 Ω | RULE: R_d ≈ 1/(3 ω_res C_f) |
| L_g, R_g | 50 µH, 5 mΩ | ASSUMPTION (stiff grid, SCR ≈ 100) |
| R1, R2 | 10 mΩ, 5 mΩ | ASSUMPTION (winding resistance) |
| f_sw | 10 kHz nominal | ASSUMPTION |
| PWM + computation delay | 1.5/f_sw | RULE |

## Control (RULE)
Current PI: K_p = ω_c L1, K_i = 0.1·K_p ω_c, ω_c = 2π·800 Hz; SRF-PLL: K_p = 2ζω_n, K_i = ω_n², ω_n = 2π·20 Hz, ζ = 0.707;
DC-link PI: K_p = ω_dc C V_dc/(1.5 V_g), K_i = 0.2 K_p ω_dc, ω_dc = 2π·15 Hz, with PV-power feed-forward; Q loop: integral, 10 Hz;
current limit 1.1 pu with active-power priority.

## Thermal FEM (COMSOL 6.x, Heat Transfer in Solids)
Quarter-symmetry model of one 62 mm module on its heatsink section (symmetry planes x = 60 mm, y = 40 mm).
Layout is REPRESENTATIVE (die placement of the commercial module is not public).
| Item | Value | Tag |
|---|---|---|
| MOSFET dies | 6 per switch, 4.04 × 6.44 × 0.18 mm | ASSUMPTION (6 × 25 mΩ dies ≈ 4.17 mΩ) |
| SBD dies | 4 per switch, a × 6.44 × 0.18 mm, a calibrated | CALIBRATED to R_th,jc,SBD = 0.076 K/W |
| Die attach / substrate solder | 80 µm / 150 µm, effective k calibrated | CALIBRATED to R_th,jc,MOS = 0.075 K/W (lumps voids and interfaces) |
| DBC | Cu 0.3 / AlN 0.635 / Cu 0.3 mm (AlN: DATASHEET insulator material) | ASSUMPTION thicknesses |
| Baseplate | Cu, 106 × 61 × 3 mm (DATASHEET: Cu baseplate, 62 mm footprint) | |
| TIM | 50 µm, k = 1 W/(m·K) | ASSUMPTION |
| Heatsink section | Al, 120 × 80 mm footprint, 12 mm base, 8 fins 2.5 × 35 mm | ASSUMPTION |
| Convection | h_nom = 40 W/(m²·K) on fin surfaces (forced air), scaled by cooling factor h_fac | ASSUMPTION |
| Materials | k [W/(m·K)], ρ [kg/m³], c_p [J/(kg·K)]: SiC 370/3210/690, Cu 400/8960/385, AlN 170/3260/740, solder 58(bulk)/7400/230, Al 200/2700/900, TIM 1/2500/1000 | handbook room-temperature values; constant properties (limitation) |
| Junction temperature | volume average over the MOSFET dies of one switch (virtual junction temperature); max also exported | |
| Case temperature | average over the baseplate bottom under the DBC | |
Heatsink sized so that T_j ≈ 115 °C at 100 kW, 40 °C ambient; the inverter must derate at high ambient
(typical of PV inverters, which derate above ≈45–50 °C).
