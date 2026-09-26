# 1D1V Vlasov–Poisson Solver using Phase Point Trajectories

A Python implementation of a **1D1V electrostatic Vlasov–Poisson solver** based on the **Phase Point Trajectory (PPT)** / characteristics method.

The solver follows discrete phase points in phase space, reconstructs the distribution function on an Eulerian grid using the **Method of Average**, solves the periodic Poisson equation using FFTs, and advances the system using a **Leapfrog drift–kick scheme**.

The project also includes diagnostics for **energy conservation, mass conservation, recurrence, computational performance, and PPC convergence**.

---


## Features

- 1D1V electrostatic Vlasov–Poisson model
- Phase Point Trajectory / characteristics formulation
- Multiple phase points per phase-space cell (PPC)
- Optional randomized phase-point jitter
- Method of Average for phase-space deposition
- Neighbor-based void filling
- Trapezoidal integration for electron density
- Periodic FFT-based Poisson solver
- Bilinear interpolation of the electric field to phase points
- Leapfrog time integration
- Kinetic, electrostatic, internal, and total energy diagnostics
- Total mass conservation monitoring
- Numerical recurrence analysis
- PPC convergence study
- Function-level runtime profiling
- Phase-space and energy diagnostics

---

## Numerical Model

The 1D1V Vlasov equation is written as

$$
\frac{\partial f}{\partial t} + v\frac{\partial f}{\partial x} - E\frac{\partial f}{\partial v} = 0
$$

The phase points follow the characteristics

$$
\frac{dx}{dt} = v
$$

and

$$
\frac{dv}{dt} = -E(x,t)
$$

The distribution value carried by each phase point remains constant along its trajectory:

$$
\frac{df}{dt} = 0
$$

The electrostatic field is obtained from the Poisson equation

$$
\frac{\partial^2 \phi}{\partial x^2} = n_e - 1
$$

with

$$
E = -\frac{\partial \phi}{\partial x}
$$

The ion background density is normalized to unity.

---

## Method of Average

The PPT method stores the distribution function on moving phase points

$$
(x_p, v_p, f_p)
$$

while the Poisson solver requires the distribution function on a fixed Eulerian grid

$$
f(x_i, v_j)
$$

This implementation reconstructs the distribution using the **Method of Average**.

Each phase point is assigned to its nearest phase-space grid node. If multiple phase points occupy the same node, their distribution values are averaged:

$$
f_{ij} = \frac{1}{N_{ij}} \sum_{p \in (i,j)} f_p
$$

This approach avoids the explicit interpolation weights required by conventional bilinear deposition.

### Void Filling

Grid nodes that receive no phase points are treated as voids.

The implementation fills these locations using neighboring grid values through a local averaging stencil. This reduces artificial discontinuities in the reconstructed distribution and improves the robustness of the subsequent density and electric-field calculation.

---

## Poisson Solver

The Poisson equation is solved spectrally using the FFT.

For periodic boundary conditions,

$$
-k^2 \phi_k = \rho_k
$$

so for nonzero Fourier modes,

$$
\phi_k = -\frac{\rho_k}{k^2}
$$

The zero Fourier mode is set to zero to fix the potential gauge.

The electric field is then obtained from

$$
E_k = -ik\phi_k
$$

The physical-space potential and electric field are recovered using the inverse FFT.

---

## Leapfrog Integration

The solver uses a staggered Leapfrog formulation.

The phase-point position is stored at integer time steps:

$$
x^n
$$

while the velocity is stored at half time steps:

$$
v^{n+1/2}
$$

Each time step consists of:

### 1. Drift

$$
x^{n+1} = x^n + \Delta t\,v^{n+1/2}
$$

### 2. Distribution Reconstruction

Reconstruct $f(x,v)$ using the Method of Average.

### 3. Density Calculation

$$
n_e(x) = \int f(x,v)\,dv
$$

### 4. Poisson Solve

Calculate $\phi$ and $E$ on the spatial grid.

### 5. Field Interpolation

Interpolate the electric field from the Eulerian grid to the phase points.

### 6. Kick

$$
v^{n+3/2} = v^{n+1/2} - \Delta t\,E_p
$$

### 7. Velocity Handling

The velocity is clipped to the predefined velocity domain.

---

## Diagnostics

The solver provides several physical and numerical diagnostics.

### Total Mass

$$
M = \int \int f(x,v)\,dx\,dv
$$

implemented numerically using the phase-space grid.

### Kinetic Energy

$$
E_k = \frac{1}{2} \int \int f(x,v)v^2\,dx\,dv
$$

### Electrostatic Energy

$$
E_E = \frac{1}{2} \int E^2(x)\,dx
$$

### Total Energy

$$
E_{\mathrm{total}} = E_k + E_E
$$

The relative energy-conservation error is calculated as

$$
Error = 100 \frac{|E(t)-E(0)|}{|E(0)|}
$$


### Internal Energy

The internal energy is calculated from the velocity variance relative to the local mean velocity.

This provides an additional diagnostic of the velocity-space distribution and numerical evolution.

---

## Runtime Profiling

A lightweight decorator-based profiler is included:

```python
@run_time
def my_function(...):
    ...
