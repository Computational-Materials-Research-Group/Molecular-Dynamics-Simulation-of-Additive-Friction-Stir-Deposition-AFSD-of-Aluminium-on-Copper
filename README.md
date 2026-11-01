# Molecular Dynamics Simulation of Additive Friction Stir Deposition (AFSD) of Aluminium on Copper


![LAMMPS](https://img.shields.io/badge/LAMMPS-MD%20Simulation-blue?style=for-the-badge)
![Process](https://img.shields.io/badge/AFSD-Solid--State%20Additive%20Manufacturing-orange?style=for-the-badge)
![Material](https://img.shields.io/badge/Al%20on%20Cu-Dissimilar%20Deposition-black?style=for-the-badge)
![Tool](https://img.shields.io/badge/Tool-Hollow%20Rotating%20W-red?style=for-the-badge)
![Potential](https://img.shields.io/badge/EAM%2Falloy-Cu--Al--W-purple?style=for-the-badge)
![OVITO](https://img.shields.io/badge/OVITO-Visualization-green?style=for-the-badge)
[![DOI](https://img.shields.io/badge/DOI-10.5281%2Fzenodo.23864143-blue?style=for-the-badge)](https://doi.org/10.5281/zenodo.23864143)
[![License: CC BY-NC 4.0](https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge)](http://creativecommons.org/licenses/by-nc/4.0/)

A fully atomistic **molecular dynamics simulation of Additive Friction Stir Deposition (AFSD)**, using LAMMPS. A square aluminium feed rod is pushed down through the bore of a hollow tungsten tool that rotates and traverses above a copper substrate. Rod atoms that leave the bore under the shoulder are released from the rigid feedstock and become free material, which the rotating shoulder forges into a deposited layer.

The simulation captures **plasticization of the protruding feedstock**, **material flow under the rotating shoulder**, **layer formation behind the traversing tool**, **Al/Cu interfacial mixing**, and **flash formation at the shoulder edge**, using an *EAM/alloy potential (CuAlW.txt)* for Cu, Al and W. The tool-material contact force, substrate temperature, and the number of deposited atoms are written every 1,000 steps, and the full trajectory is written for visualization in OVITO.

<img width="1600" height="1200" alt="afsd new" src="https://github.com/user-attachments/assets/9f117558-7fa7-4f06-95cd-3bf530eb0dc0" />

---

## Physics

The simulation captures the deposition stage of AFSD at the atomistic scale:

- Al feedstock, Cu substrate and W tool modelled with a single EAM/alloy potential containing all three elements
- Hollow tool: an annular W cylinder, with the square feed rod passing through its bore
- Tool and rigid feedstock rotate together about the tool axis (spindle) and traverse along x
- Only the feedstock is fed down through the bore; the tool keeps a fixed standoff equal to the layer thickness
- Deposition by state change: rod atoms that pass below the shoulder face leave the rigid stock and join the free (NVE) material
- Deposited atoms are relabelled as a separate atom type, so the layer can be isolated directly in OVITO
- Soft feed on a firm substrate (Al on Cu), as in real AFSD, so the feedstock deforms instead of indenting the substrate
- Simulation box is a whole number of Cu unit cells in x and y, so there is no overlap seam at the periodic boundaries
- Substrate relaxed at 300 K with an x/y barostat (z is a free surface) before deposition
- Langevin thermostat on a buried substrate layer only, so frictional heating in the deposition zone is not damped artificially

---

## Speed Regime

| Quantity                     | Real AFSD           | This Simulation               |
| ---------------------------- | ------------------- | ----------------------------- |
| Shoulder edge speed          | ~0.5 to 2 m/s       | **600 m/s** (0.2 rad/ps x 30 Å) |
| Traverse speed               | ~1 to 10 mm/s       | **20 m/s** (0.2 Å/ps)         |
| Feed speed                   | ~1 to 5 mm/s        | **20 m/s** (0.2 Å/ps)         |
| Process time                 | seconds to minutes  | **250 ps**                    |

MD process speeds are many orders of magnitude above experimental values because of the femtosecond timestep. The ratios between speeds are kept closer to real AFSD than the absolute values. Edge speeds above ~1,000 m/s (close to the shear-wave speed of Al) melt the substrate surface and drag it up the tool; keep `omega * r_tool` well below that. Check speed sensitivity before drawing quantitative conclusions.

---

## Tool and Feedstock

| Parameter              | Script variable | Default      | Role                                                       |
| ---------------------- | --------------- | ------------ | ---------------------------------------------------------- |
| Shoulder outer radius  | `r_tool`        | 30 Å         | Sets the width of the deposited track                      |
| Bore radius            | `r_inner`       | 17 Å         | Must exceed the rod corner distance (15.0 Å)               |
| Tool height            | `tool_height`   | 60 Å         | Length of the bore guiding the rod                         |
| Rod equivalent radius  | `r_rod`         | 12 Å         | Square rod of equal area, side = r_rod·√π = 21.3 Å         |
| Layer thickness        | `h_layer`       | 8 Å          | Tool standoff above the substrate                          |
| Rod tip gap            | `tip_gap`       | 2 Å          | Initial gap between rod tip and substrate (no overlap)     |
| Rod reserve            | `rod_extra`     | 60 Å         | Rod length above the tool, consumed by feeding             |
| Rotation rate          | `omega`         | 0.2 rad/ps   | Spindle speed (one revolution = 31.4 ps)                   |
| Traverse speed         | `U`             | 0.2 Å/ps     | Tool and stock speed along x                               |
| Feed speed             | `Ufeed`         | 0.2 Å/ps     | Downward speed of the stock through the bore               |

---

## Geometry

```
Side view (x-z plane through the tool axis, units: Angstrom):

z = 190 +--------------------------------------------------------+  <- box top (free)
        |                                                        |
z = 168 |                 +-------+                              |  <- rod top
        |                 |  Al   |  rigid stock (type 1)        |
        |                 | stock |  rotates + traverses + feeds |
z = 108 |     +-----------+       +-----------+                  |  <- tool top
        |     |    W      |       |    W      |                  |
        |     |  tool     |       |  tool     |  hollow tool     |
        |     | (type 3)  |       | (type 3)  |  rotates +       |
        |     |           |       |           |  traverses       |
z =  48 |     +-----------+-------+-----------+                  |  <- shoulder face
        |                 |deposit|  -> released Al (type 4)     |     (release plane)
z =  40 +--------------------------------------------------------+  <- substrate top
        |                                                        |
        |                Cu substrate (type 2)                   |
z =  16 |- - - - - - - - - - - - - - - - - - - - - - - - - - - - |
        |          Langevin layer, 300 K (6 to 16 Å)             |
z =   6 |- - - - - - - - - - - - - - - - - - - - - - - - - - - - |
        |////////////// fixed base (0 to 6 Å) //////////////////|
z =   0 +--------------------------------------------------------+
        x = 0          tool axis x = 60 -> 110            x = 242.2

x: traverse direction (periodic)
y: transverse (periodic), tool axis at y = Ly/2 = 81.3
z: build direction (fixed, non-periodic)
```

- **Box**: 242.2 x 162.7 x 190 Å = 67 x 45 Cu unit cells in x and y
- **Substrate**: Cu, FCC, a = 3.615 Å, 40 Å thick
- **Feed rod**: Al, FCC, a = 4.05 Å, 21.3 x 21.3 Å square, 126 Å long
- **Tool**: W, BCC, a = 3.165 Å, hollow cylinder, outer radius 30 Å, bore radius 17 Å, height 60 Å
- **Track**: tool axis moves from x = 60 to x = 110 Å (50 Å), at least 30 Å from the periodic boundary at the end of the run
- **Standoff**: shoulder face 8 Å above the substrate (equal to the layer thickness)

### Approximate System Size

| Component                | Atom type | Atoms (approx.) |
| ------------------------ | --------- | --------------- |
| Cu substrate             | 2 (Cu)    | ~138,700        |
| of which fixed base      | 2 (Cu)    | ~24,100         |
| W tool                   | 3 (W)     | ~7,300          |
| Al feed rod (stock)      | 1 (Al)    | ~3,300          |
| Al deposited (at start)  | 4 (Al)    | ~150            |
| **Total**                | -         | **~149,000**    |

The exact numbers are printed by `create_atoms` at the top of the log file.

---

## Atom Types

| Type | Element | Role                                                                 |
| ---- | ------- | -------------------------------------------------------------------- |
| 1    | Al      | Rigid feedstock (stock) inside and above the bore                    |
| 2    | Cu      | Substrate                                                            |
| 3    | W       | Rigid hollow tool                                                    |
| 4    | Al      | Deposited material (released stock), free dynamics                   |

Types 1 and 4 are both mapped to Al in `pair_coeff`, so they interact identically; the separate type only marks the deposited material.

---

## Simulation Phases

| Phase            | Run Steps   | Simulated Time | Description                                                              |
| ---------------- | ----------- | -------------- | ------------------------------------------------------------------------ |
| Setup            | -           | -              | Box, substrate, rod and tool created; overlapping rod atoms removed      |
| Equilibration    | 20,000      | 10 ps          | Substrate and initial rod tip at 300 K, NPT in x and y (box relaxes)     |
| Deposition       | 500,000     | 250 ps         | 5,000 segments of 100 steps: rotate, traverse, feed, release             |
| **Total MD**     | **520,000** | **260 ps**     | -                                                                        |

---

## Expected Event Timeline

The events below are the expected sequence for the default parameters. Use `v_ndep` and the trajectory of your own run for the actual timing.

| Event                                              | Time in deposition | Approx. dump frame |
| -------------------------------------------------- | ------------------ | ------------------ |
| Protruding rod tip touches and wets the substrate  | 0 to ~10 ps        | 5 to 7             |
| Tip shears and spreads under the rotating shoulder | ~10 to 30 ps       | 7 to 12            |
| Shoulder footprint fills with Al                   | ~30 to 80 ps       | 12 to 25           |
| Steady deposition, layer trails behind the tool    | ~80 to 250 ps      | 25 to 130          |
| Flash forms at the shoulder edge (if over-fed)     | any time           | -                  |
| End of run (tool axis at x = 110 Å)                | 250 ps             | 130                |

Dump frames are every 4,000 steps (2 ps). Frame 5 is the end of equilibration.

---

## Material Parameters

### Lattices and Masses

| Material | Role       | Structure | Lattice constant (Å) | Mass (g/mol) | Atom types |
| -------- | ---------- | --------- | -------------------- | ------------ | ---------- |
| Al       | Feedstock  | FCC       | 4.05                 | 26.982       | 1, 4       |
| Cu       | Substrate  | FCC       | 3.615                | 63.546       | 2          |
| W        | Tool       | BCC       | 3.165                | 183.840      | 3          |

### Process Parameters

| Parameter        | Symbol  | Value   | Unit     |
| ---------------- | ------- | ------- | -------- |
| Rotation rate    | ω       | 0.2     | rad/ps   |
| Traverse speed   | U       | 0.2     | Å/ps     |
| Feed speed       | U_f     | 0.2     | Å/ps     |
| Layer thickness  | h       | 8       | Å        |
| Track length     | L       | 50      | Å        |
| Temperature      | T       | 300     | K        |
| Timestep         | dt      | 0.0005  | ps       |
| Segment length   | -       | 100     | steps    |

### Interatomic Potential

| Parameter       | Value                                                       |
| --------------- | ----------------------------------------------------------- |
| Potential file  | `CuAlW.txt` (EAM/alloy, setfl format, elements Cu, Al, W)   |
| Potential style | `eam/alloy`                                                 |
| pair_coeff      | `* * CuAlW.txt Al Cu W Al`                                  |
| Mapping         | type 1 = Al, type 2 = Cu, type 3 = W, type 4 = Al           |

`eam/alloy` maps atom types to elements by name, so the order of the elements inside the setfl file does not matter. The tool is rigid, so the W parameters only set the tool-workpiece contact interaction.

### Boundary Conditions

| Group             | Condition                                          | Description                                   |
| ----------------- | -------------------------------------------------- | --------------------------------------------- |
| Fixed base        | `setforce 0 0 0`, v = 0                            | Bottom 6 Å of the substrate held in place     |
| Thermostat layer  | NVE + Langevin 300 K (damping 0.5 ps)              | Removes frictional heat through the substrate |
| Free material     | NVE                                                | Substrate above the base + deposited Al       |
| Spindle           | `fix move rotate` + `displace_atoms`               | Rigid tool and stock, prescribed motion       |
| x, y              | Periodic                                           | Infinite plate in the plane                   |
| z                 | Fixed (f)                                          | Free surface above the substrate              |

---

## Governing Equations

### Kinematics (per segment of N = 100 steps)

```
segment time   t_s  = N * dt                    = 0.05 ps
rotation       fix move rotate about (cx, cy), period P = 2*pi / omega
traverse       dx_s = U * t_s                    = 0.01 A   (tool and stock)
feed           dz_s = -U_f * t_s                 = -0.01 A  (stock only)
axis update    cx  <- cx + dx_s
```

Each displacement (0.01 Å) is about 1/400 of a lattice spacing, so the stepwise traverse acts as a continuous motion at U.

### Tool Speeds

```
v_edge   = omega * r_tool                         (shoulder edge)
v_corner = omega * half_side * sqrt(2)            (rod corner)
advance per revolution = U * 2*pi / omega
```

### Mass Balance

```
A_rod * U_f  ~  (2 * r_tool) * h * U * phi

A_rod = (r_rod * sqrt(pi))^2 = 452 A^2
phi   = fraction of the shoulder footprint actually filled

default: 452 * 0.2 = 90.4  vs  60 * 8 * 0.2 = 96 A^3/ps
```

Too little feed gives a thin or patchy layer; too much feed pushes Al out past the shoulder as flash.

### Deposition (Release) Criterion

```
an atom changes from stock to deposited when
    type == 1   and   z < z_shoulder = z_sub_top + h_layer

then: type 1 -> 4, removed from the rigid spindle group,
      added to the free NVE group (permanent, one way)
```

### Contact Force

```
F = sum over tool atoms i, free atoms j of f_ij        (compute group/group)

F_x, F_y : in-plane reaction (traverse resistance, torque contribution)
F_z      : axial (forging) force on the tool
1 eV/A = 1.602177 nN
```

---

## Group Definitions

```
substrate      <- Cu atoms in the substrate block (0 to 40 A)
feedrod        <- all Al atoms of the rod (static, as created)
tool           <- W atoms of the hollow tool
fixed_base     <- substrate atoms with z < 6 A
substrate_flow <- substrate - fixed_base
therm_flow     <- substrate_flow atoms with 6 < z < 16 A
released       <- rod atoms that passed below the shoulder face (grows)
stock          <- feedrod - released (rebuilt every segment)
spindle        <- tool + stock (rebuilt every segment, driven by fix move)
free           <- substrate_flow + released (grows, integrated with NVE)
```

---

## Output Files

| File                    | Frequency          | Content                                                                 |
| ----------------------- | ------------------ | ----------------------------------------------------------------------- |
| `afsd_v3.dump`          | every 4,000 steps  | id, type, x, y, z, vx, vy, vz (about 130 frames)                        |
| `afsd_v3.log`           | every 1,000 steps  | Full log with thermo output                                             |
| `afsd_v3_final.data`    | end                | Final configuration (LAMMPS data format), input for further layers      |

### Thermo Output

| Column          | Quantity                                                 |
| --------------- | -------------------------------------------------------- |
| `step`, `time`  | Timestep and simulated time (ps)                         |
| `c_Tsub`        | Temperature of the free substrate (K)                    |
| `v_ndep`        | Number of deposited Al atoms                             |
| `c_f_tool[1]`   | Force on the tool from the free material, x (eV/Å)       |
| `c_f_tool[2]`   | Force on the tool from the free material, y (eV/Å)       |
| `c_f_tool[3]`   | Force on the tool from the free material, z (eV/Å)       |
| `pxx`, `pyy`    | In-plane pressure components (bar)                       |
| `lx`, `ly`      | Box lengths (Å), change only during equilibration        |

---

## Repository Structure

```
afsd_lammps/
|
|-- afsd_deposition_v3.in      # Main LAMMPS simulation script
|-- CuAlW.txt                  # EAM/alloy potential for Cu, Al and W
|-- README.md                  # This file
|
|-- generated on run/
    |-- afsd_v3.dump
    |-- afsd_v3.log
    |-- afsd_v3_final.data
```

---

## How to Run

### Requirements

- LAMMPS (any recent version with the MANYBODY package for `pair_style eam/alloy`): <https://www.lammps.org>
- EAM/alloy potential file `CuAlW.txt` with Cu, Al and W in setfl format, in the same folder as the script
- OpenMPI for parallel runs
- OVITO for trajectory visualization: <https://www.ovito.org>

On Ubuntu or WSL:

```
sudo apt update
sudo apt install -y lammps openmpi-bin dos2unix
```

### Step 1: Choose the process parameters

Edit the variables at the top of `afsd_deposition_v3.in` (sections 1 and 4):

```
variable  r_tool    equal 30.0     # shoulder radius (A)
variable  h_layer   equal 8.0      # layer thickness (A)
variable  r_rod     equal 12.0     # rod equivalent radius (A)
variable  omega     equal 0.2      # rotation rate (rad/ps)
variable  U         equal 0.2      # traverse speed (A/ps)
variable  Ufeed     equal 0.2      # feed speed (A/ps)
variable  nseg      equal 5000     # number of 100-step segments
```

Keep the mass balance and the edge speed limit in mind when changing them.

### Step 2: Run the simulation

From WSL, with the files in `D:\extras`:

```
cd /mnt/d/extras
mpirun -np 14 lmp -in afsd_deposition_v3.in -log afsd_v3.log
```

Use the number of **physical** cores for `-np` (`lscpu` shows "Core(s) per socket" x "Socket(s)"). `nproc` counts hyperthreads and can exceed the slots Open MPI allows.

Run in the background and follow the log:

```
nohup mpirun -np 14 lmp -in afsd_deposition_v3.in -log afsd_v3.log > run.out 2>&1 &
tail -f afsd_v3.log
```

If the files were edited on Windows, convert line endings first: `dos2unix afsd_deposition_v3.in`.

Files on `/mnt/d` are accessed slowly from WSL. For faster output, copy the folder to the Linux home first (`cp -r /mnt/d/extras ~/afsd && cd ~/afsd`).

### Step 3: Visualize in OVITO

1. Open OVITO, then `File > Load File` and select `afsd_v3.dump`
2. Add **Color Coding** by `Particle Type`: Al stock (1), Cu substrate (2), W tool (3), deposited Al (4)
3. To see only the deposited layer, add **Expression selection** with `ParticleType==2 || ParticleType==3` and **Delete selected**
4. Add **Slice** (normal 0 1 0, distance = Ly/2 = 81.3, slab width about 10 Å) for the cross-section along the track
5. Colour by **Velocity Magnitude** to confirm rotation (speed increasing from the bore to the shoulder edge)
6. Use **Common Neighbor Analysis** to follow the crystal structure of the deposit and the substrate surface
7. Use **Dislocation Analysis (DXA)** on the deposited layer for dislocation density
8. Count substrate uplift with **Expression selection** `ParticleType==2 && Position.Z > 40`
9. Plot `v_ndep` and `c_f_tool[3]` from the log against time

---

## What to Look for in Results

### Plasticization of the Feedstock

The protruding rod tip loses its crystal order as it is sheared between the rotating stock above and the substrate below. Common Neighbor Analysis shows the transition from FCC to disordered or defected structure.

### Layer Formation

Deposited Al (type 4) spreads under the shoulder and is left behind as the tool moves on. The layer height should approach `h_layer`. `v_ndep` rises steadily once the footprint is filled.

### Al/Cu Interface

The deposit bonds to the Cu substrate through interfacial mixing in the first few atomic layers. A firm Cu surface with only a few displaced atoms indicates the soft-feed, firm-substrate regime of real AFSD.

### Flash

Excess material leaves the footprint at the shoulder edge. Strong flash means the feed is too high for the traverse speed.

### Tool Forces

`c_f_tool[3]` gives the forging force of the shoulder on the deposit; `c_f_tool[1]` gives the resistance to traverse. Both should settle to a fluctuating steady value during steady deposition.

---

## Common Errors and Fixes

| Error or symptom                                         | Cause                                                          | Fix                                                                            |
| -------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| `Cannot open EAM potential file CuAlW.txt`               | Potential file not in the run folder                           | Copy `CuAlW.txt` next to the input script                                      |
| `Element ... not found in EAM potential file`            | Element name in `pair_coeff` differs from the setfl header     | Match the names in `pair_coeff` to line 4 of `CuAlW.txt`                       |
| `not enough slots` from mpirun                           | `-np` larger than the number of physical cores                 | Use the physical core count, or add `--use-hwthread-cpus`                      |
| Substrate atoms explode or curl at frame 0               | Box not a whole number of substrate unit cells                 | Keep `Lx`, `Ly` as integer multiples of the substrate lattice constant         |
| Substrate climbs up the tool, no visible deposit         | Edge speed too high, or feedstock harder than substrate        | Lower `omega`; use the soft-feed, firm-substrate pairing (Al on Cu)            |
| Huge forces or temperature in the first steps            | Rod tip overlapping the substrate                              | Keep `tip_gap` and the `delete_atoms overlap` line                             |
| `Lost atoms` error                                       | Fast atoms leaving through the non-periodic z boundary         | Add `thermo_modify lost ignore`, or lower `omega`                              |
| `Neighbor list overflow`                                 | Dense contact zone                                             | Add `neigh_modify one 5000`                                                    |
| Thin or patchy layer                                     | Under-feeding                                                  | Increase `Ufeed` (for example 0.25)                                            |
| Strong flash at the shoulder edge                        | Over-feeding                                                   | Decrease `Ufeed` (for example 0.15)                                            |
| `pxx`, `pyy` of thousands of bar after equilibration     | Residual lattice strain                                        | Check the box dimensions; extend the NPT run                                   |
| `$'\r': command not found`                               | Windows line endings                                           | `dos2unix afsd_deposition_v3.in`                                               |
| `conda` MPI used instead of the system MPI               | Conda environment active                                       | `conda deactivate` before running                                              |

---

## Extending the Model

| Extension                          | What to change                                                                      |
| ---------------------------------- | ----------------------------------------------------------------------------------- |
| Similar-material deposition        | Map the rod and substrate to the same element in `pair_coeff`                       |
| Cu feed on Al substrate            | Swap the lattices, masses and element mapping (expect strong substrate uplift)      |
| Process speed study                | Vary `omega`, `U` and `Ufeed` while keeping the mass balance                        |
| Layer thickness                    | Change `h_layer` (standoff and release plane move together)                         |
| Longer track                       | Increase `nseg` and `rod_extra`; keep the tool away from the periodic boundary      |
| Induction-assisted AFSD (HHSAM)    | Add a heated free zone of rod in the lower bore with a Langevin thermostat          |
| Interlayer machining (hybrid AM)   | Read `afsd_v3_final.data`, remove the tool, plane the layer with a rigid cutter     |
| Multi-layer builds                 | Read the final data file and raise `z_sub_top` by `h_layer` for each new layer      |
| Different tool material            | Change the tool lattice and element in `pair_coeff` (the tool stays rigid)          |
| Continuum comparison               | Compare with a CalculiX thermal model with element activation of the same process  |

---

## Citation

If you use this code in your research, please cite:

```
@software{mishra_2026_afsd_lammps,
  author    = {Mishra, Akshansh},
  title     = {Molecular Dynamics Simulation of Additive Friction Stir
               Deposition (AFSD) of Aluminium on Copper},
  year      = {2026},
  publisher = {Zenodo},
  doi       = {10.5281/zenodo.23864143},
  url       = {https://doi.org/10.5281/zenodo.23864143}
}
```

Plain text citation:

> Mishra, A. (2026). *Molecular Dynamics Simulation of Additive Friction Stir Deposition (AFSD) of Aluminium on Copper* [Computer software]. Zenodo. <https://doi.org/10.5281/zenodo.23864143>

Please also cite the source of the EAM/alloy potential used in `CuAlW.txt`.

---

## Author

**Akshansh Mishra**
GitHub: <https://github.com/akshansh11>

---

## License

<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/"><img alt="Creative Commons Licence" style="border-width:0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" /></a><br />
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

You are free to:

- **Share**: copy and redistribute the material in any medium or format
- **Adapt**: remix, transform, and build upon the material

Under the following terms:

- **Attribution**: You must give appropriate credit to Akshansh Mishra and provide a link to this repository
- **NonCommercial**: You may not use the material for commercial purposes

Copyright 2026 Akshansh Mishra. All rights reserved for commercial use.
