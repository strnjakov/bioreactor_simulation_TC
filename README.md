# Taylor-Couette Bioreactor Modelling for Alternative Proteins

Bioreactors are used for a variety of innovative and classic food technologies. These machines provide cells with the nutrients they need to grow and thrive. Doing this on a small scale is easy, but scaling up the process, which is necessary to bring down the price, requires a careful balancing of many factors. Spin too slowly, and the nutrients don't reach the cells. Spin too fast, and the cells shear and die. Before building a large bioreactor, it is important to decide on many variables, the effect of which is difficult to predict in advance. A digital twin is a simulation in which these parameters can be tweaked to optimise final yield of the bioreactor.

My research project uses multi-physics computational fluid dynamics to simulate these reactors. This allows for comparatively rapid virtual testing that would otherwise be very expensive to run in a lab. I aim to develop a mathematical framework that finds the "sweet spot" for stirring speeds - optimising cell growth while protecting fragile cells from physical stress across different reactor sizes.

## Features and Highlights

* **Coupled Multi-Physics Solver:** Solves the 2D incompressible Navier-Stokes equations coupled with three advection-diffusion-reaction equations for dissolved oxygen ($O$), growth substrate ($S$), and cell biomass ($B$) in **FEniCSx**.
* **Biological growth and death simulation:** Computes the biomass growth within sufficiently nutrient-dense flow, and death in high-shear and low-nutrient locations
* **Adjustable Parameters:**

---

## Project Showcase

https://github.com/user-attachments/assets/2586c1d0-82f6-46ba-b2f3-93926b3ed180



### The Goldilocks Zone

<img width="50%" alt="image" src="https://github.com/user-attachments/assets/1de74d16-d7fb-491a-af40-547629419091" />

The model resolves the non-linear trade-off between mass transport and hydrodynamic clearance across inner cylinder velocities ($U_{\text{inner}} \in [0.02, 0.30]\,\text{m/s}$):

* **Transport-Limited Regime ($U_{\text{inner}} < 0.05\,\text{m/s}$):** Radial eddy mixing is insufficient to transport boundary oxygen across the gap ($t_{\text{diff}} \gg T_{\text{process}}$), resulting in widespread hypoxic arrest.
* **Optimal Operating Window ($U_{\text{inner}} \approx 0.05\,\text{m/s}$):** Convective mixing delivers oxygen to the expanding cell cluster within process timescales while local shear stress remains below lethal thresholds, maximizing viable biomass harvest.
* **Shear-Dominated Regime ($U_{\text{inner}} > 0.15\,\text{m/s}$):** Hydrodynamic shear near the inner cylinder accelerates quadratic cell lysis ($k_d \propto U_{\text{inner}}^2$), clearing over 40% of cumulative potential biomass.

## Mathematical Model

### Governing Equations

**Advection-Diffusion-Reaction System:**

$$\partial_t O + \mathbf{u}\cdot \nabla O = \nabla \cdot (D_{O,\text{eff}} \nabla O) - R_O$$

$$\partial_t S + \mathbf{u}\cdot \nabla S = \nabla \cdot (D_{S,\text{eff}} \nabla S) - R_S$$

$$\partial_t B + \mathbf{u}\cdot \nabla B = \nabla \cdot (D_{B,\text{eff}} \nabla B) + R_B$$

**Monod Growth Kinetics:**

$$\mu(O, S) = \mu_{\max} \left(\frac{S}{K_S + S}\right) \left(\frac{O}{K_O + O}\right)$$

$$R_B = \mu(O, S) B - k_d(\dot{\gamma}) B$$

$$R_O = Y_O \mu(O, S) B + m_O \left(\frac{O}{K_O + O}\right) B$$

$$R_S = Y_S \mu(O, S) B + m_S \left(\frac{S}{K_S + S}\right) B$$

**Shears and Diffusion**

* **Shear Rate:** $\dot{\gamma} = \sqrt{2\,\mathbf{S}:\mathbf{S}}$, where $\mathbf{S} = \frac{1}{2}(\nabla \mathbf{u} + \nabla \mathbf{u}^T)$
* **Clearance Mortality:** $k_d(\dot{\gamma}) = k_{d,0} + \alpha_{\text{shear}} \dot{\gamma}^2$
* **Eddy Diffusivity:** $D_{\text{turb}} = (C_{\text{mix}} d)^2 \dot{\gamma}$, with $D_{i,\text{eff}} = D_{i,\text{mol}} + D_{\text{turb}}$

### Finite Element Formulation

The coupled system is discretised using mixed Lagrange elements ($P_1$ for scalar transport fields, $P_2$ for fluid velocity) and stabilized using a Streamline Upwind Petrov-Galerkin (SUPG) formulation:

$$F_{\text{total}} = \sum_{i \in \{O, S, B\}} \left( F_{i, \text{Gal}} + F_{i, \text{SUPG}} \right) = 0$$

$$F_{i, \text{Gal}} = \int_{\Omega} \left( \frac{C_i - C_{i,n}}{\Delta t} v_i + (\mathbf{u} \cdot \nabla C_i) v_i + D_{i,\text{eff}} \nabla C_i \cdot \nabla v_i - R_i v_i \right) dx$$

$$F_{i, \text{SUPG}} = \int_{\Omega} \left( \frac{C_i - C_{i,n}}{\Delta t} + \mathbf{u} \cdot \nabla C_i - R_i \right) \left( \tau_{\text{SUPG}} \, \mathbf{u} \cdot \nabla v_i \right) dx$$

where $\tau_{\text{SUPG}} = \frac{h}{2 \|\mathbf{u}\|}$.
