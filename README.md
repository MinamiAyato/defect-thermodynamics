# Defect Thermodynamics: A Modular Computational Materials Project

## About This Project

This project was inspired by Dr. Seán Kavanagh's defect workflow.

The original workflow is powerful, but for beginners entering computational materials science, especially defect calculations, it can be difficult to see which skills are fundamental and which steps are simply part of a larger automated workflow.

This repository therefore tries to break the workflow down into several independent capability modules, so that beginners can practise and understand each idea more efficiently.

At the same time, this project is also a way for the author to strengthen his own understanding of computational materials science, scientific Python, and defect physics.

The author believes that the laws of nature should ultimately be simple.

> If something feels impossibly complicated, there is probably still a missing piece in our understanding, or we have added one abstraction layer too many.

So the goal here is not to make the workflow look sophisticated, but to make the underlying ideas as clear as possible.

Hopefully, this repository can be useful to other beginners as well, and we can improve together.


## Project Goals

The main goals of this project are to:

- understand how computational materials data are represented and processed in Python;
- connect crystal-structure operations with their physical meaning;
- learn how numerical convergence is evaluated;
- understand how defect structures and charge states are generated;
- explore why multiple local defect geometries must be considered;
- build toward defect thermodynamics and Fermi-level analysis.

The emphasis is on understanding the logic behind the workflow rather than treating scientific software as a black box.

---

## Project Structure

### 1. Structure File I/O

Learn how crystal structures are stored in files and represented as Python objects.

Main topics:

- manually inspect and parse a POSCAR;
- convert structure files into `pymatgen.Structure` objects;
- inspect lattice, composition, coordinates, and volume;
- write structure objects back to files.

Core idea:

**structure file → Python object → inspect or modify → write back**

---

### 2. Simulation & Numerical Reliability

Learn how to extract results from multiple calculations and determine whether a numerical parameter is sufficiently converged.

Main topics:

- locate and read multiple VASP output files;
- extract quantities such as ENCUT and final total energy;
- organise comparable calculation results;
- evaluate numerical convergence;
- balance numerical accuracy and computational cost.

Core idea:

**find calculation outputs → extract comparable data → evaluate energy changes → choose a sufficiently converged setting**

A larger numerical parameter is not automatically the best choice. The goal is to find a value beyond which further increases produce only negligible changes in the calculated result.

---

### 3. Atomic Structure & Defect Modelling

Learn how a pristine crystal is transformed into physically meaningful defect models.

Main topics:

- obtain a primitive structure from a bulk structure;
- manually construct a supercell;
- compare manual and automated supercell generation;
- generate vacancies, substitutions, and interstitials;
- generate possible defect charge states;
- use `ShakeNBreak` to generate multiple distorted starting structures;
- understand how relaxed structures are compared to identify lower-energy defect geometries.

Core idea:

**primitive structure → supercell → defect → charge state → distorted structures → lowest-energy defect geometry**

The initial defect geometry is only a starting hypothesis. Different local distortions may relax into different minima on the potential-energy surface, so several candidate structures should be explored.

---

### 4. Defect Physics & Thermodynamics

This module will connect calculated defect energies to thermodynamic behaviour.

Planned topics:

- chemical potentials;
- dielectric screening and finite-size corrections;
- defect formation energies;
- charge-transition levels;
- equilibrium defect concentrations;
- carrier concentrations;
- self-consistent Fermi-level determination.

Core idea:

**defect energetics → thermodynamic stability → concentrations → electronic behaviour**

---

## Tools

The project uses Python-based materials-science tools including:

- `pymatgen`
- `doped`
- `ShakeNBreak`
- `NumPy`
- `pandas`
- `Matplotlib`

VASP-specific input generation is not the main focus of this repository. Instead, the project concentrates on the transferable physical and computational ideas behind defect calculations.

---

## Current Progress

Completed or in progress:

- [x] project environment and repository setup
- [x] structure file parsing and `pymatgen` structure handling
- [x] ENCUT convergence analysis
- [x] primitive-cell and supercell construction
- [x] automated defect generation with `doped`
- [x] defect charge-state generation
- [ ] distorted defect structures with `ShakeNBreak`
- [ ] relaxed-structure energy comparison
- [ ] defect formation energies
- [ ] defect thermodynamics and Fermi-level analysis

---

## Learning Philosophy

The project follows a simple principle:

**understand the physical problem first, then use scientific software to solve it efficiently.**

Manual implementations are used when they help reveal the underlying logic. Once the concept is understood, validated scientific libraries are preferred for routine calculations.