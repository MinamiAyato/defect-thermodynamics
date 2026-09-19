# Synthetic MgO defect-thermodynamics dataset

This is a compact, internally consistent **training dataset** inspired by the shape of VASP workflows. Every numerical result is synthetic and every VASP-like output was written from scratch for this exercise; no full proprietary or copyrighted output is reproduced. The text files intentionally contain enough realistic clutter to require robust parsing, but are much smaller than production outputs.

## Scientific model

- Host: rocksalt MgO, 64-atom `2×2×2` conventional supercell (`Mg32O32`).
- Defect: `Mg_O`, represented as `Mg33O31`, in charge states `q = 0,+1,+2,+3,+4`.
- Energy zero for electronic thermodynamics: valence-band maximum (`VBM = 0 eV`).
- Synthetic band gap: `7.20 eV`.
- Defect formation energy convention:

  `E_f(Mg_O^q,E_F) = E_def(q) - E_bulk - μ_Mg + μ_O + q(E_F + E_VBM) + E_corr(q)`

  Here positive `n_i` means atoms added to the defect cell. For `Mg_O`, `n_Mg=+1` and `n_O=-1`, hence the chemical-potential term above.

## Directory map

```text
00_documentation/                  schema, units, manifest
01_bulk_convergence/
  encut/                           8-atom fixed-cell single points
  kpoints/                         8-atom fixed-cell single points
02_bulk_relaxed/                   final 64-atom bulk calculation
03_defect_structure_search/Mg_O/   four candidate distortions per charge state
04_final_defects/Mg_O/             selected q=0…+4 calculations
05_chemical_potentials/            elemental and competing-phase energies
06_dielectric_convergence/         dielectric tensors vs settings
07_dos/                            DOSCAR-like data and band-edge metadata
08_reference/                      expected values for tests (use only after trying)
```

## Suggested parsing exercises

1. Recursively find `OUTCAR` files and extract `SYSTEM`, `ENCUT`, k-mesh, `NIONS`, `NELECT`, final `E0`, final `TOTEN`, maximum force, ionic steps, and convergence status.
2. For ENCUT and k-point sweeps, calculate `ΔE` relative to the highest setting in meV/atom. Choose the least expensive setting within `1 meV/atom` of the reference.
3. For each charge state in the structure search, rank distortions by final `E0`. Detect the deliberately unconverged candidate and decide whether to exclude it.
4. Join final defect `metadata.json`, `OUTCAR`, `vasprun.xml`, and `correction.json` into one table.
5. Reproduce formation-energy lines under O-rich, midpoint, and Mg-rich chemical potentials. Find the lower envelope and charge-transition levels within `0–7.2 eV`.
6. Use `site_density_cm-3` and degeneracy from metadata to calculate dilute-limit defect concentrations: `c = N_sites g exp(-E_f/k_B T)`.
7. Parse the DOS and/or use the supplied effective-density-of-states model to calculate electrons and holes, then solve charge neutrality for a self-consistent Fermi level. State any approximations.

## Important details and traps

- Folder names are helpful metadata, but the authoritative charge and composition are in `metadata.json`.
- `free energy TOTEN` and extrapolated `final energy (E0)` differ slightly; use one definition consistently. The reference calculations use `E0`.
- One structure-search calculation is intentionally unconverged.
- Corrections are stored separately and must be added exactly once.
- DOS energies are already shifted so that `VBM=0`; the Fermi energy printed in the DOS header is not the thermodynamic Fermi level.
- The structure files are POSCAR/CONTCAR-style, not guaranteed to support every strict VASP reader because this is a pedagogical reduced dataset.

## Recommended first notebook

Start with `01_bulk_convergence`, write a reusable OUTCAR parser, add tests against `08_reference/expected_results.json`, and only then reuse the parser for the structure search and final defects.
