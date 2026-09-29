# Parabolic Trough Solar Collector: Axially Segmented Receiver Model (EES)

**Description:** Steady-state, one-dimensional thermal model of a small parabolic trough collector (PTC) absorber tube, written in Engineering Equation Solver (EES). The tube is divided into axial segments, each solved by an internal fixed-point procedure, so the model needs no guess values and converges reliably. The model structure is inspired by Forristall, *NREL/TP-550-34169* (2003).

- **File:** `PTC5.EES` (model version 2)
- **Tool:** Engineering Equation Solver (EES), SI units (`SI C Pa J W`)
- **Working fluid:** Water at 101.3 kPa
- **Receiver:** Bare copper tube with matte black coating (no glass envelope, no selective coating)

---

## Assumptions

- Steady state, no wind (natural convection only)
- No glass envelope around the absorber
- No tracking: sun is normal to the aperture (`K_angle = 1`)
- Matte black coating, copper tube; reflector reflectance is a placeholder (`rho_cl = 0.85`) until the aperture material is chosen
- Optical error terms from Forristall, *NREL/TP-550-34169* (2003), Table 2.4
- Aperture and absorber sizes come from a separate MATLAB sizing script
- Effective sky temperature is 8 K below ambient

## Method

The absorber is split into `N` segments along its length. The outlet of segment *i* is the inlet of segment *i+1*. Each segment is solved by the procedure `SegSolve`, which iterates the chain

`Q -> T_out -> T_m -> h -> T2 -> T3 -> losses -> new Q`

until the heat delivered to the water per unit length (`Q`, W/m) converges. Because iteration happens inside the procedure, water and air properties are never evaluated at impossible temperatures.

| Step | Model |
|---|---|
| Water-side convection | Laminar (Re < 2300): thermally developing entry-length correlation, averaged over the segment from the cumulative mean Nu. Turbulent (Re >= 2300): Gnielinski with viscosity-ratio correction (NREL eq. 2.5 and 2.6) |
| Wall conduction | Radial conduction through the copper wall (`k = 390 W/m-K`) |
| Outer convection | Natural convection to ambient air, Churchill-Chu correlation |
| Radiation | Grey-body exchange between outer wall and sky |
| Solar input | `q_si = DNI * W_aperture`, reduced by the optical efficiency chain and absorptivity |

**Unknowns per segment:** `T_out`, `T2` (inner wall), `T3` (outer wall), `Q`.

## Parameters (as in `PTC5.EES`)

| Parameter | Symbol | Value |
|---|---|---|
| Number of segments | `N` | 20 |
| Absorber inner / outer diameter | `D2` / `D3` | 2 mm / 2.2 mm |
| Aperture width | `W_aperture` | 0.2 m |
| Absorber length | `L_pipe` | 0.4 m |
| Direct normal irradiance | `DNI` | 800 W/m² |
| Water mass flow rate | `mdot` | 0.0002 kg/s |
| Water inlet temperature | `T1_in` | 25 °C |
| Ambient / sky temperature | `T6` / `T7` | 25 °C / 17 °C |
| Coating emissivity / absorptivity | `eps3_coating` / `alpha_abs` | 0.95 / 0.95 |
| Copper conductivity | `k_23` | 390 W/m-K |

**Optical efficiency terms:** shadowing 0.974, tracking error 0.994, geometry error 0.98, dirt 1.0 (clean), unaccounted losses 0.96, reflectance 0.85 (placeholder), incidence angle modifier 1.

## Key outputs

| Variable | Meaning |
|---|---|
| `T1_out` | Water outlet temperature |
| `Q_useful` | Useful heat delivered to the water (W) |
| `Q_loss` | Heat lost to surroundings (W) |
| `q_useful_avg` | Average useful heat per unit length (W/m) |
| `eta_thermal` | Thermal efficiency = `q_useful_avg / q_si` |
| `eta_optical` | Optical efficiency including absorptivity |
| `T2_max` | Hottest inner wall temperature (at the outlet) |
| `Check_global` | Energy balance check, should be ~0 |

Per-segment arrays (`T_in[i]`, `T_out[i]`, `T_m[i]`, `T2[i]`, `T3[i]`, `Q_seg[i]`, `Loss_seg[i]`, `Nu_D2[i]`, `Re_D2[i]`) are available for plotting.

## How to use

1. Open `PTC5.EES` in EES and press **F2** (Solve). No guess values are needed.
2. Check that `Check_global` is approximately 0.
3. Plot `T_out[i]`, `T_m[i]` and `T2[i]` against `x2[i]` for the axial temperature profile.
4. Vary `N` (1, 5, 10, 20, 40) and confirm that `T1_out` and `eta_thermal` stop changing (grid independence).
5. With `N = 1` the model reduces to the original single-node model.

## Expected results (default parameters)

Reproduced independently in Python with CoolProp for the same equations and inputs. EES property values may differ slightly.

| Quantity | Value |
|---|---|
| Water outlet temperature | about 77.8 °C |
| Useful heat | about 44.1 W |
| Heat loss | about 2.9 W |
| Thermal efficiency | about 69% |
| Optical efficiency | about 73.6% |
| Hottest inner wall temperature | about 90.2 °C |
| Inlet Reynolds number | about 148 (laminar) |

Grid independence check: outlet temperature changes by less than 0.02 °C between `N = 20` and `N = 40`.

## Limitations

- Reflectance is a placeholder and must be updated once the reflector material is chosen.
- No glass envelope, wind, or tracking; results represent an idealised fixed, normal-incidence case.
- Axial conduction in the wall and end losses are not modelled.
- Fluid properties are evaluated at 101.3 kPa.

## Reference

Forristall, R. *Heat Transfer Analysis and Modeling of a Parabolic Trough Solar Receiver Implemented in Engineering Equation Solver.* NREL/TP-550-34169, 2003.
