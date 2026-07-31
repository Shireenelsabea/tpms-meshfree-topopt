# Experimental validation plan: printed compression test of the optimized graded gyroid

Status: protocol v1, 2026-07-31. Companion files: `make_specimen_stl.py`,
`specimen_uniform_t0.30.stl`, `specimen_graded_opt.stl`.

## 1. Objective and the key design of the experiment

The paper's headline claim is a **ratio**, not an absolute number: at equal
volume fraction (Vf = 0.193), the optimized graded gyroid is stiffer than the
uniform gyroid in the x1 direction by **+9.1%** (voxel FEM) with the mesh-free
model predicting +17.5%. The experiment therefore compares **two specimen
groups printed in the same build** and validates the stiffness ratio

    R = E*_graded / E*_uniform

Testing the ratio (instead of absolute stiffness) cancels the dominant
experimental unknowns: base material modulus, print-process softening,
layer anisotropy, and machine compliance all appear in both numerator and
denominator. This is the same logic the paper uses when it validates design
ranking rather than absolute values.

Expected outcome bands:
- R in 1.05-1.13: confirms the FEM-predicted gain (9.1% +/- realistic scatter).
- R near 1.00: gain not realized; investigate wall fidelity at t_min regions.
- R above 1.15: suggests the DEM prediction band; unlikely but informative.

## 2. Specimens

| | uniform | graded (optimized) |
|---|---|---|
| geometry | gyroid sheet, t = 0.30 | t(x) with theta* from E3, delta_t0 = -0.0094 |
| tessellation | 4 x 4 x 4 unit cells | same |
| cell size | 20 mm | same |
| bounding box | 80 x 80 x 80 mm | same |
| volume fraction | 0.1930 | 0.1929 (0.02% apart) |
| solid volume | 98.8 cm^3 | 98.8 cm^3 |
| wall thickness min/med/max | 1.12 / 1.30 / 1.39 mm | 0.44 / 1.04 / 2.70 mm |

Both STLs carry a 1.2 mm ridge along one edge parallel to **x1** (the
optimized loading direction). The ridge is identical on both specimens, adds
under 0.1% volume, and removes any ambiguity about orientation after
depowdering. The graded field is periodic, so the tessellation has no
thickness discontinuities at cell boundaries.

Replicates: **n = 5 per group, 10 specimens total**, all in a single build,
placement randomized across the platform (do not cluster one group in one
corner; thermal history varies across the bed). If build volume forces two
builds, put equal numbers of both groups in each build.

## 3. Printing

Recommended processes, in order of preference:

1. **MJF or SLS, PA12.** Self-supporting TPMS, no support removal, quasi-
   isotropic material. Minimum wall of the graded specimen is 0.44 mm, which
   is at the capability edge of powder-bed polymer processes (typical
   guideline 0.5 mm). Options: (a) accept and quantify (Section 4), or
   (b) regenerate at L_CELL = 25 mm (min wall 0.56 mm, 100 mm cube).
2. **SLA/DLP resin.** Resolves 0.44 mm walls comfortably; use a rigid resin
   and identical post-cure for all 10 specimens; drain holes are not needed
   (gyroid pores are fully interconnected).
3. Metal LPBF only if a metals lab is the target venue; not required for the
   validation logic.

Same build orientation for all specimens (x1 in-plane, recorded), same batch
of powder/resin, same slicer profile. The STL files are ~250 MB (5.2 M
triangles at 64 samples per cell); regenerate at other resolutions or cell
sizes by editing two constants in `make_specimen_stl.py`.

## 4. Metrology before testing (this is what reviewers will ask for)

1. **Mass** of every specimen (+/- 0.01 g). Iso-volume is the constraint of
   the whole optimization; demonstrate it experimentally:
   mass_graded / mass_uniform should be 1.000 +/- 0.01. Report the actual
   relative density vs the design value 0.193 (powder-bed prints typically
   come out a few percent heavy at thin walls).
2. **Outer dimensions** with calipers (shrinkage/scale factor).
3. **Wall thickness spot checks**: optical microscope or micro-CT on one
   sacrificial specimen per group. Critical for the graded specimen at the
   t_min = 0.12 regions; if printed walls there are systematically thicker
   than designed, the realized geometry is between "graded" and "uniform"
   and R will shrink accordingly. A micro-CT of one specimen per group,
   registered against the analytic level set, is the single highest-value
   measurement in this plan and directly reuses the paper's voxel pipeline
   (the CT voxel field can be run through the same code_aster KUBC analysis:
   as-printed FEM vs as-designed FEM separates geometry error from model
   error).

## 5. Mechanical test protocol

Standard: **ISO 13314** (compression of cellular materials) adapted to
polymer AM lattices; ASTM D1621 as the polymer-foam alternative.

- Quasi-static compression along **x1** (the marked axis), nominal strain
  rate 1e-3 /s (4.8 mm/min for the 80 mm specimen).
- Hardened, ground platens; thin PTFE film or grease at both interfaces to
  reduce friction confinement.
- **Strain from optics, not crosshead**: DIC on one lateral face, or a video
  extensometer between platen-adjacent gauge marks. Machine compliance is
  the classic artifact in lattice modulus data.
- Loading program per ISO 13314: preload to ~0.05 MPa nominal stress, then
  **three load-unload cycles** between approximately 20% and 70% of the
  expected proportional limit; take the **unloading modulus** of the final
  cycle as E*. Unloading modulus is far less sensitive to early local
  plasticity and seating than the initial loading slope.
- Continue the final loading to 10-20% strain to record the plateau; energy
  absorption is a free secondary result (TPMS literature values it), but it
  is not part of the validation claim.

## 6. Data analysis

1. E* per specimen from the unloading slope (stress = F/A0 with
   A0 = 80 x 80 mm nominal; consistent for both groups, so the ratio is
   unaffected by the area convention).
2. R = mean(E*_graded) / mean(E*_uniform) with a 95% CI via Welch's t /
   propagation, or bootstrap over the 5 x 5 specimen pairs.
3. Power check: with typical AM stiffness scatter of CV 3-5%, n = 5 per
   group detects a 9% mean difference at alpha = 0.05 with power > 0.9.
   If pilot scatter exceeds CV 6%, add specimens before concluding.
4. Compare against three model values, all at Vf = 0.193:
   - FEM (as-designed): R = 0.11699 / 0.10725 = **1.091**
   - DEM (mesh-free): R = 0.1338 / 0.1139 = 1.175
   - FEM (as-printed CT geometry, if Section 4.3 is done): the tightest
     comparison, isolating solver error from print error.
5. Report absolute E* too (with the measured base-material modulus from
   printed dogbones if available), but frame the validation on R.

## 7. Known gaps between model and experiment (state them, do not hide them)

- **Boundary conditions**: the paper's C1111 is a single-cell KUBC (affine
  displacement) bound; the experiment is a finite 4x4x4 array between
  platens (mixed conditions, free lateral faces). The comparison of ratios
  largely cancels this, but for the paper's discussion: KUBC is an upper
  bound, and the half-cell boundary layer at free faces softens both groups
  similarly. A finite-specimen FE model of the full 4x4x4 array (same voxel
  pipeline, platen-contact BCs) closes this gap completely if a reviewer
  presses; it is a large but tractable solve.
- **Linear vs real material**: E* is extracted in the small-strain elastic
  range precisely so the linear-elastic model applies.
- **Direction**: the gain is directional (C1111). Optionally test 3
  additional specimens per group along x2 to show the contrast between the
  optimized direction and a non-optimized one; a directional signature
  matching the model is stronger evidence than a single ratio.

## 8. Effort and cost estimate

10-13 polymer prints of ~100 g each, one build day on an MJF/SLS bureau or
two days on a desktop SLA, one micro-CT session (optional but recommended),
and one day on a universal testing machine with DIC. No custom fixtures.

## 9. Track B: FDM dog-bone demonstrators (accessible, desktop-printer route)

Companion files: `make_dogbone_stl.py`, `dogbone_uniform.stl`,
`dogbone_optimized.stl`, `dogbone_preview.png`.

Where Track A (compression cubes, Sections 2-6) is the rigorous
quantitative test, Track B is a demonstrator that any FDM printer can
produce and any tensile machine can pull, and it showcases a point the
paper can make in one sentence: the specimen itself is generated by the
paper's own thickness-grading mechanism, with no CAD. Ramping t(x) above
max|F| = 1.5 turns the level set solid, so the solid grip ends, the
filleted transition, and the TPMS gauge all come from the single analytic
equation phi = max(phi_outline, |F(kx)| - t_spec(x)).

Specimen pair (identical outline, gauge volume fractions 0.1926 vs 0.1929):

| | value |
|---|---|
| overall / thickness | 160 x 45 x 15 mm plate dog-bone |
| gauge section | 60 x 30 x 15 mm = 4 x 2 x 1 cells of 15 mm |
| fillet radius | 25 mm; t(x) ramps to solid over the 30-40 mm band |
| gauge lattice | uniform t = 0.30 vs optimized graded theta* |
| tensile axis | x1, the optimized C1111 direction |
| mass (PLA) | ~77 g each |

Printing (FDM, PLA, 0.4 mm nozzle): print flat, 0.15-0.2 mm layers,
100% infill with 2+ perimeters (the lattice walls themselves become
perimeter paths), no supports needed inside the gauge (the gyroid is
self-supporting; it is literally the geometry behind slicers' "gyroid
infill"). Same spool, same profile, same plate position class for both.
Caveat to record: the thinnest graded walls (t_min regions, ~0.34 mm
designed at this cell size) will print at nozzle width; weigh both parts
and report measured masses next to the designed 0.17% volume difference.

Test: tension at 2 mm/min with an extensometer or DIC over the central
50 mm of the gauge; three load-unload cycles in the elastic range;
report the gauge-section stiffness ratio graded/uniform, n >= 3 pairs.
Expectations and caveats: the single-cell thickness and free lateral
surfaces make this a structural comparison rather than a homogenization
measurement, so the measured ratio will sit below the bulk KUBC value of
1.091; the model claim being demonstrated is the *ranking* (graded
stiffer than uniform at equal mass) plus a visible, printable artifact
of mesh-free geometry-parameter optimization. For the journal-grade
number, use Track A.

## 10. What goes into the paper

A new "Experimental validation" subsection: specimen table (Section 2),
mass/iso-volume check, E* per group with CIs, measured R vs FEM 1.091 and
DEM 1.175, and the CT wall-fidelity figure. This elevates the work above the
entire physics-informed TO literature cited in the manuscript, none of which
reports a physical specimen.
