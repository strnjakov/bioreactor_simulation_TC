# Taylor-Couette Bioreactor Modelling for Alternative Proteins

Bioreactors are used for a variety of innovative and classic food technologies. These machines provide cells with the nutrients they need to grow and thrive. Doing this on a small scale is easy, but scaling up the process, which is necessary to bring down the price, requires a careful balancing of many factors. Spin too slowly, and the nutrients don't reach the cells. Spin too fast, and the cells shear and die. Before building a large bioreactor, it is important to decide on many variables, the effect of which is difficult to predict in advance. A digital twin is a simulation in which these parameters can be tweaked in order to optimise the output yield of the bioreactor.

My research project uses multi-physics computational fluid dynamics to simulate these reactors. This allows for comparatively rapid virtual testing that would otherwise be very expensive to run in a lab. I aim to develop a mathematical framework that finds the "sweet spot" for stirring speeds - optimising cell growth while protecting fragile cells from physical stress across different reactor sizes.

## Features and Highlights

* **Coupled Multi-Physics Solver:** Solves the 2D incompressible Navier-Stokes equations coupled with three advection-diffusion-reaction equations for dissolved oxygen ($O$), growth substrate ($S$), and cell biomass ($B$) in **FEniCSx**.
* **Biological growth and death simulation:** Computes the biomass growth within sufficiently nutrient-dense flow, and death in high-shear and low-nutrient locations
* **Adjustable Parameters:**

---

## Simulation Showcase

https://github.com/user-attachments/assets/2586c1d0-82f6-46ba-b2f3-93926b3ed180

Four views of the same test run of the same bioreactor. In the top right you can see the **Fluid Velocity**. The viscosity and timescale is boosted in this animation by a few orders of magnitude, so the spin-up time is very quick, but you can still see it happen. Currently, the fluid simulation is executed entirely and the output is fed to the biomass simulation.

In the top left, the **Biomass** is displayed. It is stirred by the (precomputed) fluid flow. It proliferates wherever there is plentiful Substrate and Oxygen, however, it struggles to proliferate towards the centre of the cylinder, where strong shear forces are present. 

In the bottom you can see the **Substrate** and the **Oxygen**. They are both consumed when Biomass grows. Oxygen is replenished through the inner boundary whereas in this simulation, Substrate is not replenished.

The rotation speed can be adjusted. Increasing it increases the effective diffusion coefficient of the Oxygen (so it is replaced more easily), but also increases the shear forces that tear through the cells, leading to an interesting optimisation problem.

### The Goldilocks Zone

<img width="50%" alt="image" src="https://github.com/user-attachments/assets/1de74d16-d7fb-491a-af40-547629419091" />

In the above diagram the final Biomass yield is shown across a range of rotation velocities. We can see there is a tradeoff between mass transport and shear stress that must be balanced. Here the sweet spot appears to be at around 0.05 m/s. Below this value, the oxygen and substrate simply don't reach the cells fast enough, and they are starved of the nutrients they need. Above the critical value, shear forces simply kill too many cells, and the population plummets.

## Mathematical Model

### **Advection-Diffusion-Reaction System:**

$$\partial_t O + \mathbf{u}\cdot \nabla O = \nabla \cdot (D_{O,\text{eff}} \nabla O) - R_O$$

$$\partial_t S + \mathbf{u}\cdot \nabla S = \nabla \cdot (D_{S,\text{eff}} \nabla S) - R_S$$

$$\partial_t B + \mathbf{u}\cdot \nabla B = \nabla \cdot (D_{B,\text{eff}} \nabla B) + R_B$$

### **Growth Equations:**

$$\mu(O, S) = \mu_{\max} \left(\frac{S}{K_S + S}\right) \left(\frac{O}{K_O + O}\right)$$

$$R_B = \mu(O, S) B - k_d(\dot{\gamma}) B$$

$$R_O = Y_O \mu(O, S) B + m_O \left(\frac{O}{K_O + O}\right) B$$

$$R_S = Y_S \mu(O, S) B + m_S \left(\frac{S}{K_S + S}\right) B$$

### **Shear and Diffusion**

* **Shear Rate:** $\dot{\gamma} = \sqrt{2\,\mathbf{S}:\mathbf{S}}$, where $\mathbf{S} = \frac{1}{2}(\nabla \mathbf{u} + \nabla \mathbf{u}^T)$
* **Clearance Mortality:** $k_d(\dot{\gamma}) = k_{d,0} + \alpha_{\text{shear}} \dot{\gamma}^2$
* **Eddy Diffusivity:** $D_{\text{turb}} = (C_{\text{mix}} d)^2 \dot{\gamma}$, with $D_{i,\text{eff}} = D_{i,\text{mol}} + D_{\text{turb}}$

### Finite Element Formulation

The coupled system is discretised using mixed Lagrange elements ($P_1$ for scalar transport fields, $P_2$ for fluid velocity) and stabilised using a Streamline Upwind Petrov-Galerkin (SUPG) formulation:

$$F_{\text{total}} = \sum_{i \in \{O, S, B\}} \left( F_{i, \text{Gal}} + F_{i, \text{SUPG}} \right) = 0$$

$$F_{i, \text{Gal}} = \int_{\Omega} \left( \frac{C_i - C_{i,n}}{\Delta t} v_i + (\mathbf{u} \cdot \nabla C_i) v_i + D_{i,\text{eff}} \nabla C_i \cdot \nabla v_i - R_i v_i \right) dx$$

$$F_{i, \text{SUPG}} = \int_{\Omega} \left( \frac{C_i - C_{i,n}}{\Delta t} + \mathbf{u} \cdot \nabla C_i - R_i \right) \left( \tau_{\text{SUPG}} \, \mathbf{u} \cdot \nabla v_i \right) dx$$

where $\tau_{\text{SUPG}} = \frac{h}{2 \|\mathbf{u}\|}$.
