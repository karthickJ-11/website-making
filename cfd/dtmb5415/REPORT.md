# DTMB 5415 calm-water resistance: STAR-CCM+ setup report

- **Case:** DTMB 5415 surface combatant, model scale 1:51.1, Froude number 0.39, calm water
- **Method:** STAR-CCM+ RANS with an overset mesh, DFBI free sinkage and trim, and VOF free surface
- **Reference run:** the same model computed earlier in IITM-RANS3D (refined-region cells 0.03 m lateral, 0.01 m vertical)
- **Prepared:** October 2026

This report collects the final STAR-CCM+ setup for the case, with every number derived from the hull dimensions and the speed. The mesh matches the earlier IITM-RANS3D run so the two solvers can be compared directly. Values marked ⚑ come from published DTMB 5415 particulars as recalled, not checked against the source documents. Values marked *(assumed)* are reasonable choices that still need the matching RANS3D input (see [Section 13](#13-assumptions-and-open-items)).

Companion files in this folder:
- [`DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html`](DTMB5415_1-51.1_Fn039_STARCCM_setup_guide.html): ITTC-based guide with a finer STAR-only grid family (SST k-ω, 6.8 ms time step).
- [`DTMB5415_STARCCM_replicate_IITM-RANS3D.html`](DTMB5415_STARCCM_replicate_IITM-RANS3D.html): first version of the RANS3D-matched sheet.

Where this report and those files differ, this report holds the later decisions: standard k-ε, y⁺ = 30, a 0.001 s time step with at most 10 inner iterations per step (the companion files use 20), 20 s of physical time, and a background overlap box matched to the overset cells.

---

## Contents

1. [Final settings at a glance](#1-final-settings-at-a-glance)
2. [Model particulars](#2-model-particulars)
3. [Flow parameters](#3-flow-parameters)
4. [Computational domain and boundary conditions](#4-computational-domain-and-boundary-conditions)
5. [Mesh](#5-mesh)
6. [Prism layers for y⁺ = 30](#6-prism-layers-for-y--30)
7. [Physics models](#7-physics-models)
8. [Time step](#8-time-step)
9. [Inner iterations and residual targets](#9-inner-iterations-and-residual-targets)
10. [DFBI motion](#10-dfbi-motion)
11. [Run length, monitoring and averaging](#11-run-length-monitoring-and-averaging)
12. [Resistance reduction and verification (ITTC)](#12-resistance-reduction-and-verification-ittc)
13. [Assumptions and open items](#13-assumptions-and-open-items)
14. [References](#14-references)

---

## 1. Final settings at a glance

| Item | Setting |
|---|---|
| Model length L<sub>PP</sub> | 2.779 m (142.0 m / 51.1) |
| Speed | 2.036 m/s (Fn 0.39) |
| Reynolds number | 4.97 × 10⁶ |
| Refined free-surface cells | 30 mm lateral (x, y), 10 mm vertical (z), matching IITM-RANS3D |
| Overset matching | background overlap box of 30 mm cells around the overset region |
| Hull surface cells | 15 mm target; 5 mm minimum where the hull crosses the free surface |
| Turbulence model | Standard K-Epsilon Two-Layer, Two-Layer All y⁺ wall treatment |
| Target y⁺ | 30 |
| Prism layers | first layer 0.886 mm, stretching 1.2, 10 layers, total 23.0 mm |
| Free surface | VOF, HRIC, flat VOF wave |
| Motion | DFBI, free heave and pitch; release 1.0 s, ramp 3.0 s |
| Time step | 0.001 s, 2nd-order implicit (Courant number 0.068 on 30 mm cells) |
| Inner iterations | until all residuals < 1 × 10⁻⁴, at most 10 per time step |
| Physical time | 20 s (20,000 time steps, about 15 flow-throughs) |
| Averaging window | a whole number of R<sub>T</sub> oscillation periods at the end of the run, about 2 × 5.2 s (≈ 9.6 to 20 s) |

---

## 2. Model particulars

The model dimensions are the full-scale DTMB 5415 values divided by the scale ratio λ = 51.1.

| Quantity | Full scale | Model 1:51.1 | Note |
|---|---|---|---|
| L<sub>PP</sub> | 142.0 m | **2.779 m** | |
| Beam B (waterline) | 19.06 m ⚑ | 0.373 m | |
| Draft T (even keel) | 6.15 m ⚑ | 0.1204 m | initial hull position |
| Displacement | 8424.4 t ⚑ (sea water) | ∇ = 0.06160 m³, mass 61.54 kg | fresh water, ρ = 999.1 kg/m³ |
| Wetted surface S (static, bare hull) | 2949.5 m² ⚑ | 1.1296 m² | use the CAD value at T if it differs |
| LCG | 0.683% L<sub>PP</sub> aft of midship ⚑ | 1.370 m forward of AP | set LCG = LCB from the hydrostatics |
| KG | 7.555 m ⚑ | 0.1478 m | |
| Pitch radius of gyration k<sub>yy</sub> | 0.25 L<sub>PP</sub> | 0.695 m, I<sub>yy</sub> = 29.70 kg·m² | standard assumption |

> **Scale check.** The published 5415 models are INSEAN 2340 (1:24.824), IIHR/DTMB 5512 (1:46.588) and FORCE (1:35.48). A 1:51.1 model isn't among the scales I know of. Confirm that the geometry file has L<sub>PP</sub> = 2.779 m before meshing. Every mesh size below scales with L.

Model mass is the full-scale displaced volume (8424.4 t / 1.025 t/m³ = 8218.9 m³) divided by λ³ and multiplied by fresh-water density. The model floats at the full-scale draft in fresh water.

---

## 3. Flow parameters

Water at 15 °C (ITTC 7.5-02-01-03): fresh ρ = 999.1 kg/m³, ν = 1.1386 × 10⁻⁶ m²/s; sea ν = 1.1892 × 10⁻⁶ m²/s.

| Quantity | Formula | Value |
|---|---|---|
| Model speed | V = Fn √(g L) = 0.39 √(9.81 × 2.779) | **2.0363 m/s** |
| Reynolds number | Re = V L / ν | **4.970 × 10⁶** |
| ITTC-1957 friction line | C<sub>F</sub> = 0.075 / (log₁₀Re − 2)² | **3.400 × 10⁻³** |
| Transverse Kelvin wavelength | λ<sub>w</sub> = 2π Fn² L | **2.656 m** (0.956 L) |
| Flow-through time | L / V | **1.365 s** |
| Turbulent boundary-layer thickness at AP | δ ≈ 0.37 L Re<sup>−0.2</sup> (flat plate) | ≈ 47 mm |
| Full-scale speed | V<sub>S</sub> = V √λ | 14.56 m/s (28.3 kn) |
| Full-scale Reynolds number | V<sub>S</sub> L<sub>S</sub> / ν<sub>sea</sub> | 1.738 × 10⁹ |

---

## 4. Computational domain and boundary conditions

The domain is a half domain with a symmetry plane on the centreplane. That is valid because only heave and pitch are free. The extents follow the ranges in ITTC 7.5-03-02-03: inlet 1–2 L, outlet 3–5 L, side 1.5–2 L, bottom 1–2 L.

| Boundary | Distance | Metres | Condition |
|---|---|---|---|
| Inlet (ahead of FP) | 1.5 L | 4.17 m | Velocity inlet; velocity and volume fraction from the flat VOF wave |
| Outlet (aft of AP) | 3.0 L | 8.34 m | Pressure outlet; hydrostatic pressure from the flat VOF wave |
| Side (from centreplane) | 2.0 L | 5.56 m | Velocity inlet |
| Bottom (below calm water) | 1.5 L | 4.17 m | Velocity inlet; depth Froude number V/√(g h) = 0.32, so deep water |
| Top (above calm water) | 0.5 L | 1.39 m | Velocity inlet |
| Centreplane | | | Symmetry plane, in the background and overset regions |
| Hull, transom, deck | | | No-slip wall, in the overset region |

**Wave damping:** VOF wave damping on the outlet and side boundaries, damping length 3.0 m (about 1.1 λ<sub>w</sub>), vertical damping on, default coefficients. VOF wave forcing towards the flat wave over the same length is an alternative.

These extents are *(assumed)*. Replace them with the RANS3D domain if it differs.

---

## 5. Mesh

### 5.1 Refined free-surface region (matched to IITM-RANS3D)

| Zone | Extent | Δx = Δy | Δz |
|---|---|---|---|
| **Refined free-surface region** (background and overset, identical) | 0.2 L (0.56 m) ahead of FP to 1.5 L (4.17 m) aft of AP; half-width at least 0.54 m alongside the hull (covers the overlap box), widening aft of the transom at 22°; z = −0.06 to +0.08 m *(assumed)* | **30 mm** | **10 mm** |
| **Overlap box** (background) | encloses the overset box with 0.15 m margin on every side: 0.43 m ahead of FP to 0.43 m aft of the transom; centreplane to 0.54 m; z = −0.42 m to 0.30 m above the deck | **30 mm** | **30 mm** (10 mm inside the free-surface band) |
| Global free-surface band | whole domain, z = −0.06 to +0.06 m | 60 mm | 20 mm |
| Transition boxes | one and two box widths around the refined region and overlap box | 120 / 240 mm | 40 / 80 mm |
| Base size / far-field maximum | whole domain | 120 / 480 mm | 120 / 480 mm |

Resolution of the Kelvin wave: λ<sub>w</sub> / Δx = 2.656 / 0.030 = **89 cells per transverse wavelength**, above the usual minimum of 80. Divergent waves are shorter (λ = λ<sub>w</sub> cos²θ): near the cusp line, λ ≈ 1.77 m gets about 59 cells. Expect some damping of the divergent waves in the outer wave cuts. The cell aspect ratio is Δx/Δz = 3.

The 22° half-angle is the Kelvin angle of 19.47° plus a margin. The 0.15 m overlap margin is the 0.12 m (4-cell) clearance plus 0.03 m for sinkage and trim. Only the 30 mm and 10 mm sizes come from RANS3D. The zone extents and the coarser sizes are *(assumed)*.

### 5.2 Overset region

| Item | Value |
|---|---|
| Overset box | 0.1 L (0.28 m) ahead of FP to 0.1 L aft of the transom; centreplane to 0.5 B + 0.20 m (≈ 0.39 m); 0.15 m below the keel to 0.15 m above the deck |
| Volume cells outside the free-surface band | 30 mm |
| Free-surface band inside the overset | 30 × 30 × 10 mm, identical to the background |
| Hull surface target | 15 mm *(assumed)* |
| Hull surface minimum | 3 mm below the waterline (sonar dome); **5 mm** where the hull crosses the free surface (stem, transom edge), so the Courant number there stays at or below 0.41 *(assumed)* |

### 5.3 Overset rules

- **Matching sizes:** cells on both sides of the overlap must be the same size in every direction, including the 10 mm vertical size. This is why the background uses exactly 30 × 30 × 10 mm in the free-surface band and 30 mm isotropic cells in the overlap box around the overset region.
- **Clearance:** keep at least 4 cells between the top of the prism layers and the overset boundary, and at least 4 overlapping background cells. With 30 mm cells that is about 0.12 m. Check every face of the box:
  - side: (0.20 − 0.023) / 0.030 = 5.9 cells
  - bottom: (0.15 − 0.023) / 0.030 = 4.2 cells
  - top: 0.15 / 0.030 = 5.0 cells, with no prism layers on the deck
- **Coverage:** the background overlap box must cover the overset box at every position it can reach. A 0.5° trim moves the box ends by about ±15 mm vertically, plus the sinkage. The 0.15 m overlap margin allows for this.
- **Interface:** use one overset interface between the background and overset regions with linear interpolation. Check the Overset Cell Status scalar: the acceptor cells must form a closed layer, and no inactive cells may touch the hull.

### 5.4 Free-surface smearing with 10 mm vertical cells

HRIC spreads the interface over about 2 to 3 cells, so with 10 mm vertical cells it is 20 to 30 mm thick. To limit this, set the VOF sharpening factor to 0.1. With the 0.001 s time step the free-surface Courant number is 0.068 in the 30 mm cells, 0.14 in the 15 mm hull cells and 0.41 in the 5 mm cells at the stem and transom edge. All of these are below the HRIC limit of 0.5. Wave crests right at the hull may come out slightly lower than on a finer STAR-CCM+ grid. That comes from matching the RANS3D resolution and is not a setup error.

---

## 6. Prism layers for y⁺ = 30

### 6.1 First layer height

Use the flat-plate skin-friction estimate at the hull Reynolds number.

| Step | Formula | Value |
|---|---|---|
| 1. Reynolds number | Re = V L / ν | 4.970 × 10⁶ |
| 2. Skin friction | C<sub>f</sub> = (2 log₁₀Re − 0.65)<sup>−2.3</sup> | 2.870 × 10⁻³ |
| 3. Wall shear stress | τ<sub>w</sub> = ½ ρ V² C<sub>f</sub> = 0.5 × 999.1 × 2.0363² × 2.870 × 10⁻³ | 5.945 Pa |
| 4. Friction velocity | u<sub>τ</sub> = √(τ<sub>w</sub> / ρ) | 0.0771 m/s |
| 5. Cell-centre height | y = y⁺ ν / u<sub>τ</sub> = 30 × 1.1386 × 10⁻⁶ / 0.0771 | 0.443 mm |
| 6. **First layer height** | Δy₁ = 2 y | **0.886 mm** |

STAR-CCM+ reports y⁺ at the centre of the wall cell, so the first layer is twice the cell-centre height.

### 6.2 Layer stack

| STAR-CCM+ setting | Value |
|---|---|
| Prism Layer Near Wall Thickness | **0.886 mm** |
| Prism Layer Stretching | **1.2** |
| Number of Prism Layers | **10** |
| Prism Layer Total Thickness | **23.0 mm** = 0.886 × (1.2¹⁰ − 1) / 0.2 |
| Last (outermost) layer | 4.57 mm = 0.886 × 1.2⁹ |

The total is about half the 47 mm boundary-layer thickness at the stern, which is typical with wall functions. If the step from the 4.6 mm last layer to the 15 mm hull cells looks abrupt, use 12 layers. That gives 35.1 mm total with a 6.6 mm last layer.

For reference at the same speed: y⁺ = 50 gives a first layer of 1.48 mm, and y⁺ = 100 gives 2.95 mm.

### 6.3 Checking y⁺

After about 3 s of simulated time, plot the Wall Y+ scalar on the hull. Aim for 30 to 100 over most of the hull. Local values fall below 30 near the stem and at the bilge, which the All y⁺ treatment handles. If most of the hull reads below 30, increase the first layer to about 1.5 mm (y⁺ ≈ 50) and remesh.

---

## 7. Physics models

| Model | Setting |
|---|---|
| Space and time | Three dimensional; Implicit Unsteady |
| Multiphase | Eulerian Multiphase, Volume of Fluid. Water: 999.1 kg/m³, 1.1375 × 10⁻³ Pa·s. Air (15 °C): 1.225 kg/m³, 1.79 × 10⁻⁵ Pa·s. |
| VOF | HRIC; sharpening factor 0.1; default Courant limits (0.5 / 1) |
| VOF wave | Flat wave; current 2.0363 m/s along the ship axis; calm water level at z = 0 |
| Flow solver | Segregated flow, 2nd-order convection, constant density per phase |
| Viscous regime | Turbulent, RANS |
| **Turbulence** | **Standard K-Epsilon Two-Layer**, Two-Layer All y⁺ wall treatment. Inlet turbulence intensity 1%, turbulent viscosity ratio 10. |
| Gravity | On, −9.81 m/s² in z |
| Motion | DFBI Rotation and Translation, applied to the overset region |
| Initial conditions | Velocity, volume fraction and pressure from the flat VOF wave (hydrostatic pressure) |
| Under-relaxation | Velocity 0.8, pressure 0.2, volume fraction 0.9, turbulence 0.8 |

**Why All y⁺ with standard k-ε:** y⁺ = 30 is the lowest value where wall functions are valid. The High y⁺ wall treatment is inaccurate wherever y⁺ falls below 30, and some hull areas always do. The two-layer All y⁺ treatment blends between resolved and wall-function behaviour, so it stays valid there.

**Changing from SST k-ω:** clear the solution and start from the initial conditions. That way the k-ε run has its own start-up and body-release history, and it can be compared cleanly with the SST k-ω and RANS3D runs.

**Known limits of standard k-ε:** it over-predicts turbulent kinetic energy at the bow stagnation point and is less accurate for the stern and transom flow than SST k-ω. Total resistance usually differs little between the two models, but compare them on this case before relying on either.

---

## 8. Time step

### 8.1 How it is chosen

The time step must satisfy three conditions; the smallest wins.

| Criterion | Rule | Limit for this case |
|---|---|---|
| ITTC 7.5-03-02-03 (resistance) | Δt = 0.005 to 0.01 × L/V | 6.8 to 13.6 ms |
| Free-surface Courant number | C = V Δt / Δx ≤ 0.5 (sharp HRIC interface), on the **smallest** cells the free surface crosses | 30 mm cells: Δt ≤ 7.4 ms; 15 mm hull cells: Δt ≤ 3.7 ms; 5 mm stem and transom cells: Δt ≤ 1.2 ms |
| Body motion | at least 100 time steps per heave/pitch period (estimated period about 0.8 s ⚑) | Δt ≤ 8 ms |

The heave period was estimated as 2π √((m + a₃₃) / (ρ g A<sub>WP</sub>)), with added mass a₃₃ ≈ m and a waterplane coefficient of 0.78. That makes it a rough estimate only.

### 8.2 Chosen value

**Δt = 0.001 s**, 2nd-order temporal discretization.

| Check | Value |
|---|---|
| Free-surface Courant number | **0.068** on 30 mm cells (2.0363 × 0.001 / 0.030); 0.14 on 15 mm hull cells; 0.41 on 5 mm stem and transom cells |
| In flow-through units | 0.00073 L/V, finer than the ITTC range |
| Time steps per heave period | about 780 |
| Time steps for 20 s | 20,000 |

The binding limit is the Courant number in the 5 mm cells where the hull crosses the free surface, which needs Δt ≤ 1.2 ms. A time step of 0.001 s satisfies it. In the 30 mm refined region it is much finer than needed, so the grid, not the time step, limits the accuracy there.

Monitor the maximum convective Courant number in cells containing the interface (volume fraction between 0.05 and 0.95). It should stay below 0.5.

### 8.3 Why not a Courant number of 10⁻⁴ or 10⁻³

| Courant target (30 mm cells) | Time step | Time steps for 20 s | Verdict |
|---|---|---|---|
| 10⁻⁴ | 1.47 × 10⁻⁶ s | about 13.6 million | impractical |
| 10⁻³ | 1.47 × 10⁻⁵ s | about 1.36 million | impractical |
| 0.068 | 1.0 × 10⁻³ s | 20,000 | **chosen** |
| 0.46 | 6.8 × 10⁻³ s | about 2,930 | ITTC value, but too large for the 15 mm and 5 mm hull cells (Courant 0.93 and 2.8) |

Compared with 6.8 ms, the 0.001 s step takes 6.8 times as many time steps. The total cost is about 3.4 to 6.8 times as much, depending on how many inner iterations each step needs.

Courant numbers below about 0.2 don't improve calm-water resistance. The 10⁻⁴ target belongs to the residuals (Section 9), not to the Courant number.

---

## 9. Inner iterations and residual targets

Each time step iterates until every residual reaches 10⁻⁴, with a cap.

| Stopping criterion | Setting |
|---|---|
| Continuity, X-, Y- and Z-momentum, Tke, Tdr, water volume fraction | Minimum Limit = **1 × 10⁻⁴**; Inner Iteration Criterion ticked; Logical Rule **And** |
| Maximum Inner Iterations | **10**; Logical Rule **Or** (the cap) |
| Maximum Physical Time | 20 s; remove the Maximum Steps criterion or set it high |
| Residual normalization | default (normalized over the first iterations), so 10⁻⁴ is relative to the start-up residuals |

Tdr is the ε residual and replaces Sdr (ω) from the SST setup.

- With a 0.001 s time step each step starts close to the converged state, so the 10⁻⁴ target is usually reached in fewer than 10 iterations.
- Some steps hitting the cap right after the body release (1 to 4 s) is normal.
- If most steps hit the cap with continuity stuck near 10⁻³, look for mesh problems first, such as overset orphan cells or poor prism-layer quality. Raising the cap doesn't fix that.

---

## 10. DFBI motion

| Item | Setting |
|---|---|
| Degrees of freedom | translation in z (heave) and rotation about y (pitch) only |
| Mass (half model) | **30.77 kg** (full model 61.54 kg) |
| I<sub>yy</sub> (half model) | **14.85 kg·m²** (full 29.70 kg·m²); I<sub>xx</sub> and I<sub>zz</sub> any positive value, since those motions are fixed |
| Centre of gravity | LCG 1.370 m forward of AP ⚑, KG 0.148 m ⚑, y = 0 |
| Initial position | even keel at T = 0.1204 m |
| Release time | **1.0 s** (about 0.73 L/V) |
| Ramp time | **3.0 s** (about 2.2 L/V) |
| 6-DOF solver | body updated every inner iteration |

Enter half values because of the symmetry plane. In the first steps the hydrostatic heave force on the half hull should balance the half weight, 30.77 × 9.81 = **301.9 N**. A mismatch means the mass, draft or water density is wrong.

The release and ramp times are in seconds, so they don't change with the time step.

**Towing point:** with surge fixed, the force that holds the model (the equivalent of the tow force) acts through the centre of gravity. In a towing tank it acts at the tow point. The resulting pitch-moment difference is small; to match a tank test closely, add an external force equal to R<sub>T</sub> at the tow-point height.

---

## 11. Run length, monitoring and averaging

| Item | Value |
|---|---|
| Physical time | **20 s** = 14.7 flow-throughs (L/V = 1.365 s) |
| Start-up | ramp complete at 4 s. Vortical disturbances leave the domain by about 8 s (4 s + 8.34 m / V). Transient waves move aft at about V/2 and pass the outlet by about 12 s. |
| Start-up wave transient | a resistance oscillation with period 8πV/g = **5.2 s** that decays slowly, because part of the start-up wave system travels with the hull |
| **Averaging window** | a whole number of R<sub>T</sub> oscillation periods at the end of the run. Two periods are about 10.4 s, so about 9.6 to 20 s; use the oscillation period seen in the R<sub>T</sub> plot |
| Convergence check | over the window, the mean of R<sub>T</sub> does not drift and the oscillation is < 1% of the mean; sinkage and trim flat |
| If still drifting | continue the saved simulation to 25 or 30 s, average over whole periods again; don't restart |

**Monitors to set up:**
- total resistance R<sub>T</sub> on the hull, with the pressure and shear parts separate
- CG translation in z (sinkage) and pitch angle (trim)
- residuals
- Wall Y+ on the hull
- Overset Cell Status
- free surface: the VOF = 0.5 isosurface coloured by height

---

## 12. Resistance reduction and verification (ITTC)

### 12.1 Model-scale coefficients

- C<sub>T</sub> = R<sub>T</sub> / (½ ρ V² S), with S = 1.1296 m² static wetted surface for the full hull. The half model gives half of R<sub>T</sub>, so double it.
- C<sub>F</sub> (ITTC-1957) = 3.400 × 10⁻³ at Re = 4.97 × 10⁶.
- Form factor (1 + k): from a double-body (no free surface) run at the same Reynolds number, or a Prohaska plot.
- Wave resistance C<sub>W</sub> = C<sub>T</sub> − (1 + k) C<sub>F</sub>.

### 12.2 Full-scale extrapolation (ITTC 1978 method, 7.5-02-03-01.4)

C<sub>TS</sub> = (1 + k) C<sub>FS</sub> + ΔC<sub>F</sub> + C<sub>A</sub> + C<sub>W</sub> + C<sub>AAS</sub>

| Term | Value |
|---|---|
| Re<sub>S</sub> | 1.738 × 10⁹ |
| C<sub>FS</sub> | 1.431 × 10⁻³ |
| ΔC<sub>F</sub> = 0.044 [(k<sub>S</sub>/L)<sup>1/3</sup> − 10 Re<sub>S</sub><sup>−1/3</sup>] + 0.000125, k<sub>S</sub> = 150 µm | 2.07 × 10⁻⁴ |

### 12.3 Comparing with RANS3D and test data

- **Against IITM-RANS3D:** compare C<sub>T</sub> with pressure and friction separated, sinkage, trim, and wave cuts at y/L = 0.082, 0.172 and 0.324. Use the same S, ρ and V in both. A difference in friction points to the wall treatment or turbulence model; a difference in pressure points to free-surface resolution or method.
- **Against published tank data:** INSEAN 2340 and IIHR 5512 were tested at larger scales. Their resistance, sinkage and trim were measured over a Froude-number range that includes 0.39, so read those values at Fn 0.39 directly from the curves. Wave cuts and local flow data exist only at Fn 0.28 and 0.41. Compare the residuary coefficient C<sub>R</sub> = C<sub>T</sub> − (1 + k) C<sub>F</sub>, sinkage and trim rather than C<sub>T</sub>, because C<sub>T</sub> depends on the Reynolds number.

### 12.4 Grid and time-step study (ITTC 7.5-03-01-01)

ITTC 7.5-03-01-01 changes one input at a time and holds the others fixed. So run a grid study at a fixed time step, then a time-step study on the medium grid.

**Grid study:** three grids with refinement ratio r = √2, the RANS3D grid as the medium, all at Δt = 0.71 ms. That keeps the Courant number below 0.5 on the fine grid's smallest hull cells (3.5 mm). Scale every size control by r, including the hull surface sizes and the overlap box. Keep the first prism layer fixed so y⁺ stays at 30, and refine tangentially. That makes the study not strictly systematic near the wall, which should be stated when reporting it.

| Grid | Refined Δx = Δy | Refined Δz | Hull surface target / waterline minimum | Δt |
|---|---|---|---|---|
| Coarse | 42.4 mm | 14.1 mm | 21.2 / 7.1 mm | 0.71 ms |
| **Medium (RANS3D match)** | **30 mm** | **10 mm** | **15 / 5 mm** | 0.71 ms |
| Fine | 21.2 mm | 7.07 mm | 10.6 / 3.5 mm | 0.71 ms |

**Time-step study:** medium grid at Δt = 1.41, 1.00 and 0.71 ms.

For each result S (C<sub>T</sub>, sinkage, trim), with grids 1 = fine, 2 = medium, 3 = coarse:

1. ε₂₁ = S₂ − S₁ and ε₃₂ = S₃ − S₂. The convergence ratio R<sub>G</sub> = ε₂₁ / ε₃₂ is monotonic for 0 < R<sub>G</sub> < 1, oscillatory for R<sub>G</sub> < 0 and divergent for R<sub>G</sub> > 1.
2. If monotonic: p<sub>G</sub> = ln(ε₃₂/ε₂₁) / ln r; δ\*<sub>RE</sub> = ε₂₁ / (r<sup>p<sub>G</sub></sup> − 1); C<sub>G</sub> = (r<sup>p<sub>G</sub></sup> − 1) / (r<sup>p<sub>est</sub></sup> − 1), with p<sub>est</sub> = 2.
3. U<sub>G</sub> = [9.6 (1 − C<sub>G</sub>)² + 1.1] |δ\*<sub>RE</sub>| if |1 − C<sub>G</sub>| < 0.125, otherwise [2 |1 − C<sub>G</sub>| + 1] |δ\*<sub>RE</sub>|. If oscillatory, U<sub>G</sub> = ½ (S<sub>max</sub> − S<sub>min</sub>).
4. Apply the same steps to the time-step study to get U<sub>T</sub>. Then U<sub>SN</sub> = √(U<sub>I</sub>² + U<sub>G</sub>² + U<sub>T</sub>²), where U<sub>I</sub> is the iterative uncertainty from the averaging window.
5. Validation: E = D − S. The result is validated if |E| < U<sub>V</sub> = √(U<sub>D</sub>² + U<sub>SN</sub>²).

---

## 13. Assumptions and open items

These RANS3D inputs decide how closely the STAR-CCM+ case matches the earlier run.

| Item | Current choice | Why it matters |
|---|---|---|
| Motion in RANS3D (free or fixed) | free heave and pitch | if RANS3D was fixed, drop DFBI and the overset region and mesh the hull at the RANS3D attitude |
| Extents of the 30 × 10 mm region | Kelvin wedge, z = −0.06 to +0.08 m | both solvers must resolve the same part of the wave field |
| RANS3D near-wall grid and wall treatment | y⁺ = 30, wall functions | changes the friction resistance directly |
| RANS3D turbulence model and free-surface method | standard k-ε; VOF (STAR has no level set) | |
| RANS3D time step | 0.001 s | |
| Domain extents | 1.5 L / 3 L / 2 L / 1.5 L / 0.5 L | |
| Hull particulars marked ⚑ | published values as recalled | check against the geometry's hydrostatics |
| Model scale 1:51.1 | as given | not a published 5415 model scale |

---

## 14. References

1. ITTC 7.5-03-02-03, *Practical Guidelines for Ship CFD Applications*: domain extents, time step 0.005–0.01 L/V.
2. ITTC 7.5-03-01-01 (2017), *Uncertainty Analysis in CFD Verification and Validation*: R<sub>G</sub>, p<sub>G</sub>, C<sub>G</sub> and U<sub>G</sub>.
3. ITTC 7.5-02-02-01, *Resistance Tests*.
4. ITTC 7.5-02-03-01.4, *1978 ITTC Performance Prediction Method*: ΔC<sub>F</sub> formula.
5. ITTC 7.5-02-01-03, *Fresh Water and Seawater Properties*.
6. Hino, T., Stern, F., Larsson, L., Visonneau, M., Hirata, N., Kim, J. (eds.), *Numerical Ship Hydrodynamics: An Assessment of the Tokyo 2015 Workshop*, Springer, 2020.
7. Larsson, L., Stern, F., Visonneau, M. (eds.), *Numerical Ship Hydrodynamics: An Assessment of the Gothenburg 2010 Workshop*, Springer, 2014.
8. Olivieri, A., Pistani, F., Avanzini, A., Stern, F., Penna, R., *Towing Tank Experiments of Resistance, Sinkage and Trim, Boundary Layer, Wake, and Free Surface Flow Around a Naval Combatant INSEAN 2340 Model*, IIHR Report 421, 2001.
9. Longo, J., Stern, F., *Uncertainty Assessment for Towing Tank Tests with Example for Surface Combatant DTMB Model 5415*, Journal of Ship Research 49(1), 2005.
10. SIMMAN 2008 / 2014 workshop, DTMB 5415 hull particulars (source of the ⚑ values).
11. Siemens Digital Industries Software, *Simcenter STAR-CCM+ User Guide*: VOF, overset mesh, DFBI and wall treatment. Cells-per-wavelength and aspect-ratio rules are vendor guidance, not ITTC requirements.
12. Schlichting, H., *Boundary-Layer Theory*: skin-friction correlation C<sub>f</sub> = (2 log₁₀Re − 0.65)<sup>−2.3</sup>.
