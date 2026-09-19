# MgO 缺陷热力学项目路线图

这条路线按真实计算工作流组织。每一阶段都会同时产出一个 notebook、可复用函数和最小测试；后面的模块复用前面的代码，而不是重新抄一遍。

## Phase 0 — 数据与项目地图

**目标：**理解数据目录、能量参考、单位、命名规则和形成能约定。

**产出：**项目 README、环境文件、数据清单；能够解释 `E0`、`TOTEN`、`q`、`E_corr`、`μ_i` 和 `E_F` 分别来自哪里。

## Phase 1 — Bulk convergence 与第一个 OUTCAR parser

**Notebook：**`01_bulk_convergence.ipynb`

- 用 `pathlib` 扫描 ENCUT 和 k-point 目录
- 用正则表达式提取 `ENCUT / NIONS / k-point mesh / E0 / convergence`
- 计算相对最高精度参考值的 `ΔE`（meV/atom）
- 按 `1 meV/atom` 阈值选择最低成本设置
- 画两张收敛图并讨论“收敛到什么量”

**语法重点：**函数、字典、正则、列表、循环、异常、DataFrame、绘图。

## Phase 2 — Relaxed bulk 与结构检查

**Notebook：**`02_bulk_relaxation_and_structure.ipynb`

- 读取 POSCAR/CONTCAR-style 文件
- 检查元素数、原子数、晶格常数、体积和组分
- 比较初末结构，检查能量与最大力
- 把通用解析逻辑迁移到 `src/mgo_workflow/parsers.py`

## Phase 3 — 缺陷结构搜索

**Notebook：**`03_defect_structure_search.ipynb`

- 扫描 `Mg_O/q*/distortion*`
- 合并目录名、metadata 和 OUTCAR 结果
- 排除未收敛计算并按电荷态寻找最低能构型
- 画相对能量图，记录每个电荷态选中的结构

## Phase 4 — Final defect dataset

**Notebook：**`04_build_defect_dataset.ipynb`

- 解析 q=0…+4 的最终 OUTCAR、XML、metadata 和 correction
- 建立一行一个电荷态的 tidy DataFrame
- 检查 `NELECT` 与电荷、组分与缺陷类型是否一致
- 输出可复用 CSV/Parquet

## Phase 5 — Competing phases 与化学势

**Notebook：**`05_chemical_potentials.ipynb`

- 从相能量构造元素参考和形成焓
- 写出 MgO 稳定性等式及竞争相不等式
- 检查 O-rich、midpoint、Mg-rich 条件
- 可视化允许的化学势区间

## Phase 6 — Dielectric convergence 与电荷修正

**Notebook：**`06_dielectric_and_corrections.ipynb`

- 解析电子、离子和总介电张量
- 检查 ENCUT/k-mesh 收敛
- 理解 correction 文件结构与“只加一次”原则

## Phase 7 — Formation energies 与 transition levels

**Notebook：**`07_formation_energies.ipynb`

- 实现形成能函数和 Fermi-level sweep
- 比较三种生长条件
- 求所有直线交点、下包络与稳定电荷态
- 提取带隙内电荷跃迁能级

## Phase 8 — Defect concentrations

**Notebook：**`08_defect_concentrations.ipynb`

- 实现 `c = N_sites g exp(-E_f/kBT)`
- 分析温度、生长条件和费米能级影响
- 处理指数下溢与对数浓度

## Phase 9 — DOS、载流子与自洽费米能级

**Notebook：**`09_self_consistent_fermi_level.ipynb`

- 解析 DOSCAR-lite 和带边
- 计算 `n(E_F,T)`、`p(E_F,T)`
- 构造总电荷中性函数
- 用数值求根得到自洽 `E_F`

## Phase 10 — 工程化与 GitHub 发布

- 把成熟函数放入 `src/mgo_workflow/`
- 用合成参考答案建立单元测试
- 增加命令行入口、结果缓存和错误信息
- 完成方法说明、流程图、示例图和可复现实验环境

## 最终完成标准

给定同一格式的新数据目录，项目可以从原始 VASP-like 文件自动生成：收敛报告、结构搜索结果、缺陷表、形成能图、跃迁能级、浓度以及自洽费米能级，并且关键结果有测试保护。

# MgO Defect Thermodynamics Workflow

## Project Aim

Build a reproducible MgO defect workflow based on the methodology used in Seán Kavanagh’s `doped` and ShakeNBreak tutorials.

The project follows the complete path from host-structure preparation to self-consistent defect thermodynamics.

## Methodological Basis

Main tools:

- `pymatgen` for crystal structures and VASP file handling;
- `doped` for defect generation and defect thermodynamics;
- ShakeNBreak for defect structure searching;
- VASP-style data for electronic-structure calculations;
- Python for parsing, validation, analysis, and plotting.

Reference documentation:

- [doped defect-generation tutorial](https://doped.readthedocs.io/en/2.3.0/generation_tutorial.html)
- [ShakeNBreak workflow](https://shakenbreak.readthedocs.io/en/latest/ShakeNBreak_Example_Workflow.html)

## Complete Workflow

```text
Host structure
→ bulk convergence
→ bulk relaxation
→ doped defect generation
→ ShakeNBreak structure search
→ final VASP calculations
→ result parsing
→ thermodynamic inputs
→ formation energies
→ defect concentrations
→ self-consistent Fermi level
```

---

## Module 1 — Host Structure Preparation

### Purpose

Prepare a reliable MgO host structure for bulk calculations.

### Workflow

```text
crystallographic data
→ pymatgen Structure
→ primitive/conventional cell
→ bulk POSCAR
```

### Exercises

1. Create the MgO host as a `pymatgen` `Structure`.
2. Inspect its composition, lattice, volume, and symmetry.
3. Generate primitive and conventional standard cells.
4. Select and document the cell used for bulk calculations.
5. Write the selected structure as a VASP POSCAR.
6. Read the POSCAR back into Python and validate it.

### Output

```text
initial bulk POSCAR
host-structure summary
```

---

## Module 2 — Bulk Convergence and Relaxation

### Purpose

Determine reliable VASP settings and obtain the relaxed host structure required by `doped`.

### Workflow

```text
initial bulk structure
→ ENCUT convergence
→ k-point convergence
→ bulk relaxation
→ relaxed primitive host
```

### Exercises

1. Parse energies from the ENCUT calculations.
2. Calculate energy differences in meV/atom.
3. Select a converged ENCUT.
4. Parse the k-point calculations.
5. Select a converged k-point mesh.
6. Parse the relaxed bulk result.
7. Check the final energy, forces, lattice, and convergence.
8. Prepare the relaxed primitive structure for `doped`.

### Output

```text
converged VASP settings
relaxed primitive host structure
```

---

## Module 3 — Defect Generation with doped

### Purpose

Generate intrinsic MgO defects using the relaxed host structure.

### Workflow

```text
relaxed primitive host
→ doped DefectsGenerator
→ defect enumeration
→ defect supercells
→ charge states
```

### Exercises

1. Initialise `doped.generation.DefectsGenerator`.
2. Inspect the generated vacancies.
3. Inspect substitutions and antisites.
4. Locate the `Mg_O` defect.
5. Inspect its Wyckoff site and multiplicity.
6. Inspect the suggested charge states.
7. Inspect the generated bulk and defect supercells.
8. Export the unperturbed defect structures and metadata.

### Output

```text
DefectsGenerator object
Mg_O defect entries
unperturbed defect supercells
charge-state metadata
```

---

## Module 4 — Structure Search with ShakeNBreak

### Purpose

Search for lower-energy and symmetry-broken defect structures.

### Workflow

```text
doped defects
→ ShakeNBreak Distortions
→ bond distortions
→ atomic rattling
→ VASP relaxations
→ ground-state structure
```

### Exercises

1. Pass the `doped` defects to ShakeNBreak.
2. Inspect the proposed bond distortions.
3. Generate rattled structures.
4. Write the distorted VASP input directories.
5. Parse the relaxation energies.
6. Compare all distortions for each charge state.
7. Identify the lowest-energy structure.
8. Save the selected ground-state structure.

### Output

```text
distorted defect structures
energy-versus-distortion data
ground-state defect structure for each charge
```

---

## Module 5 — Parse Final VASP Results

### Purpose

Convert the final raw calculation files into structured Python data.

### Input Files

```text
OUTCAR
vasprun.xml
CONTCAR
metadata
```

### Exercises

1. Extract final total energies.
2. Check electronic and ionic convergence.
3. Extract `NELECT` and charge information.
4. Read the final relaxed structures.
5. Combine the VASP results with defect metadata.
6. Build a defect DataFrame.
7. Validate charge states, compositions, and selected structures.

### Output

```text
structured defect dataset
one row per defect charge state
```

---

## Module 6 — Chemical Potentials

### Purpose

Determine the allowed chemical potentials for MgO stability.

### Workflow

```text
elemental reference energies
+ competing-phase energies
→ formation enthalpies
→ stability constraints
→ Mg-rich/O-rich conditions
```

### Exercises

1. Parse elemental reference energies.
2. Parse MgO and competing-phase energies.
3. Calculate formation enthalpies.
4. Construct the chemical-potential constraints.
5. Determine Mg-rich and O-rich limits.
6. Check whether each growth condition satisfies all constraints.

### Output

```text
allowed chemical-potential region
Mg-rich condition
O-rich condition
```

---

## Module 7 — Dielectric Constant, Corrections, and DOS

### Purpose

Prepare the remaining inputs required for charged-defect thermodynamics.

### Workflow

```text
dielectric calculations
→ converged dielectric constant
→ charge corrections

bulk electronic structure
→ VBM
→ CBM
→ band gap
→ DOS
```

### Exercises

1. Parse the electronic dielectric tensor.
2. Parse the ionic dielectric tensor.
3. Test dielectric convergence.
4. Select the dielectric constant used for corrections.
5. Read the defect correction energies.
6. Parse the DOS data.
7. Identify the VBM, CBM, and band gap.
8. Prepare the carrier-concentration inputs.

### Output

```text
dielectric constant
charge corrections
VBM and CBM
band gap
DOS data
```

---

## Module 8 — Formation Energies and Transition Levels

### Purpose

Calculate defect stability across the band gap.

### Formation-Energy Expression

$$
E_f(D^q,E_F)
=
E_{\mathrm{def}}^q
-
E_{\mathrm{bulk}}
-
\sum_i n_i\mu_i
+
q(E_F+E_{\mathrm{VBM}})
+
E_{\mathrm{corr}}
$$

### Exercises

1. Assemble every term in the formation-energy expression.
2. Calculate formation energies at the VBM.
3. Sweep the Fermi level across the band gap.
4. Plot all charge-state formation-energy lines.
5. Calculate the lower envelope.
6. Determine the stable charge state at each Fermi level.
7. Identify the thermodynamic charge-transition levels.
8. Compare Mg-rich and O-rich conditions.

### Output

```text
formation-energy diagram
stable charge states
charge-transition levels
```

---

## Module 9 — Concentrations and Self-Consistent Fermi Level

### Purpose

Calculate equilibrium defect and carrier populations.

### Defect Concentration

$$
c(D^q)
=
N_{\mathrm{sites}}g
\exp\left(
-\frac{E_f(D^q)}{k_{\mathrm B}T}
\right)
$$

### Charge Neutrality

$$
\sum_{D,q}q\,c(D^q)
+
p(E_F)
-
n(E_F)
=
0
$$

### Exercises

1. Calculate defect concentrations.
2. Include site density and degeneracy.
3. Study the temperature dependence.
4. Calculate electron concentrations.
5. Calculate hole concentrations.
6. Construct the charge-neutrality function.
7. Solve for the self-consistent Fermi level.
8. Compare growth conditions and temperatures.
9. Report the dominant defects and carriers.

### Output

```text
defect concentrations
electron and hole concentrations
self-consistent Fermi level
dominant defect populations
```

---

## Final Project Output

The completed workflow should convert raw structure and VASP-style data into:

```text
converged bulk settings
relaxed host structure
generated defects
ground-state defect structures
structured calculation data
chemical-potential limits
formation-energy diagrams
transition levels
defect concentrations
self-consistent Fermi level
carrier concentrations
```

Each module should contain:

- a clear research question;
- a short physics description;
- numbered goals;
- matching exercises;
- explicit inputs and outputs;
- validation checks;
- reusable Python functions.

