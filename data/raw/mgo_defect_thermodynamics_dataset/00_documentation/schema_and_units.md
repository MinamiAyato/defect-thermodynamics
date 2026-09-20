# Schema and units

| Quantity | Unit | Primary source |
|---|---|---|
| total energies | eV/cell | final `energy (E0)` in OUTCAR |
| forces | eV/Å | OUTCAR |
| lattice vectors/positions | Å / fractional | POSCAR or CONTCAR |
| dielectric tensor | dimensionless | dielectric OUTCAR |
| DOS | states/eV/cell | DOSCAR-lite |
| carrier/defect concentrations | cm⁻³ | derived |
| chemical potentials | eV/atom | CSV/JSON |

Charge convention: positive `q` means electrons were removed. In these files, neutral `Mg33O31` has `NELECT=640`; therefore `NELECT=640-q`.

The JSON metadata uses explicit signed integers for charge. Directory labels (`q+2`) should be treated as identifiers, not parsed as the sole source of truth.
