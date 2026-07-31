# Mesh-free topology optimization of TPMS metamaterials

Code, verification data, and reproduction scripts for the manuscript:

> A. El Sabea, *Mesh-free topology optimization of triply periodic minimal
> surface metamaterials by deep energy minimization on analytic level sets*,
> submitted to Computer Methods in Applied Mechanics and Engineering, 2026.

The pipeline optimizes the thickness field of a gyroid sheet network with no
mesh at any stage: the geometry is an analytic level set
`phi(x) = |F(kx)| - t(x)` with exact derivatives, linear elasticity is solved
by physics-informed deep energy minimization on Monte Carlo collocation, and
the TPMS grading parameters are optimized directly. Every stage is verified
against voxel FEM (code_aster).

## Layout

```
research/
  tpms_levelset.py        analytic TPMS level-set kernel (gyroid/primitive/diamond)
                          run as a script for the N1 self-tests (gradients,
                          Hessians, volume fractions vs voxel ground truth)
  dem_cantilever.py       N2 gate: DEM solver vs Q4 FEM + Timoshenko benchmarks
  *_report.md             verification reports (N1/N2, N3, E2, E3)
  n3/
    dem_gyroid.py         single-thickness DEM homogenization (KUBC C1111)
    e3_conditional.py     conditional model u(x, t) over t in [0.15, 0.50]
    e3_optimize.py        thickness-field optimization (12-mode Fourier basis)
    make_mesh.py          voxel HEXA8 .mail generator for code_aster
    make_mesh_graded.py   same, for the optimized graded field
    homog.comm            code_aster KUBC homogenization command file
    study_*.export        code_aster run definitions
    out_*.mess            code_aster outputs (grep C1111_KUBC)
    *.pt                  trained network checkpoints (reproduce audits
                          without retraining)
    *.json                audits, optimization results, Vf tables
paper/
  make_figs_v2.py         regenerates all manuscript figures from the data
validation/
  make_specimen_stl.py    printable 4x4x4 compression specimens (uniform vs
                          optimized graded, iso-volume) via marching cubes
  EXPERIMENTAL_VALIDATION.md   full compression-test protocol (ISO 13314)
```

## Requirements

Python 3.11+, `pip install -r requirements.txt` (numpy, torch, matplotlib,
scikit-image). The FEM references need Docker with `simvia/code_aster:stable`.

## Reproducing the key numbers

```bash
# N1: geometry-kernel self-tests (three families)
python research/tpms_levelset.py gyroid

# N3/E2: DEM homogenization audit at a given thickness (uses checkpoint)
python research/n3/dem_gyroid.py 0 0.3

# FEM reference (Docker), e.g. t = 0.3 at 64^3:
python research/n3/make_mesh.py 0.3 64
docker run --rm -v "$PWD/research/n3:/shared" simvia/code_aster:stable \
  bash -lc "source /opt/activate.sh && run_aster /shared/study_N64.export"
grep C1111_KUBC research/n3/out_N64.mess

# E3: optimization case study (conditional checkpoint e3_cond.pt)
python research/n3/e3_optimize.py 300

# Manuscript figures
python paper/make_figs_v2.py

# Printable validation specimens (STL, ~250 MB each)
python validation/make_specimen_stl.py
```

Large voxel meshes (`*.mail`, up to ~80 MB) are not tracked; `make_mesh.py`
regenerates them deterministically. All reported stiffness values are
float64 audits with Monte Carlo standard errors; see the manuscript
appendices for the complete hyperparameter table.

## License

MIT, see `LICENSE`.
