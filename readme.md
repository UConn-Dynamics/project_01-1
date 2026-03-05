# **ME 5180: Advanced Dynamics | Project 01 | Group 06**

# Group Members:
[Christian DiPietrantonio](mailto:hwp25002@uconn.edu)
[Nikolas Bajramoski](mailto:nikolas.bajramoski@uconn.edu)

# Project Overview: 

In this project, a pendulum is attached to a spinning frame. The frame has dimensions, 

$h_1 = 0.2~m$

$w_1 = 0.1~m$

and the pendulum length is $L=0.15$ m with a $m=0.1$ kg point mass at the end of the system. The pendulum swings in the $x'$ - $z'$ plane as it rotates at a constant speed, $\Omega$

![Spinning pendulum with rotating  and fixed coordinate systems.](https://raw.githubusercontent.com/cooperrc/me5180-project_01/refs/heads/main/spinning_pendulum.svg)

Your team's goal is to 

- build the equations of motion using Lagrange and least action $L=T-V$
- solve for the motion for a slow rotation speed and a fast rotation speed
- visualize the solution with plots and animations

# Running the Program:

From the project directory, run:

```bash
julia main.jl 
```

All source code is in the `src/` directory.

# Project Structure

The repository is organized into a small set of Julia source files and output folders:

- `main.jl`  
  Main driver script. Defines the physical constants, initial conditions, and simulation cases for slow and fast frame rotation. It runs the solver and generates all plots.

- `src/derive_equations.jl`  
  Uses `ModelingToolkit` to derive the equation of motion symbolically from the Lagrangian formulation.

- `src/physics.jl`  
  Stores the hand-derived equation of motion, the symbolic equation converted to a numerical function, and a helper function that computes the difference between the symbolic and hand-derived accelerations.

- `src/simulate.jl`  
  Defines the state-space form of the pendulum dynamics and solves the ODE numerically using `OrdinaryDiffEq`.

- `src/visualize.jl`  
  Converts the solution for `θ(t)` into Cartesian coordinates and generates trajectory data for plotting.

- `results/`  
  Contains the generated plots for angle versus time, 3D trajectories, and the comparison between symbolic and hand-derived angular acceleration.

- `notes/` and `archive/`  
  Used for supporting work, intermediate derivations, and older project material.

# Results
(Include Plots, Animations, Comparison of hand derrived eqns to those derived using Julia)

The system was simulated for two frame rotation speeds:

- slow rotation: `Ω = 1.0 rad/s`
- fast rotation: `Ω = 12.0 rad/s`

Each case was run for two initial conditions:

- `θ(0) = 0.0 rad`, `θ̇(0) = 0.0`
- `θ(0) = 0.05 rad`, `θ̇(0) = 0.0`

The code generates three main categories of results:

1. *Angle vs. time plots*  
   These show how the pendulum angle evolves for slow and fast rotation speeds. Comparing the two angular speed cases makes it clear how the rotating frame changes the motion and equilibrium behavior of the pendulum.

2. *3D trajectory plots*  
   These visualize the mass motion in inertial coordinates. Because the pivot is rotating about the vertical axis, the pendulum mass traces a 3D path rather than oscillating in a fixed plane.

3. *Symbolic vs. hand-derived equation comparison*  
   The code computes the difference between the symbolic angular acceleration and the hand-derived angular acceleration:
   \[
   \Delta \ddot{\theta}(t)=\ddot{\theta}_{symbolic}(t)-\ddot{\theta}_{manual}(t)
   \]
   This difference should remain at or extremely near zero throughout the simulation, confirming that the Julia symbolic derivation matches the hand calculation.

Overall, the results show that increasing the frame rotation speed significantly changes the pendulum motion. At low angular speed, gravity dominates and the motion behaves more like a conventional pendulum with a rotating support. At high angular speed, the rotational terms become much more important and strongly affect the pendulum’s effective equilibrium and trajectory.


# Analysis:

To solve this problem, we can define a reference frame that rotates with the pivot about the z-axis. At time t, the rotating reference frame rotates by $\phi(t) = \Omega t$ relative to the inertial frame. Let's say we have some vector $\vec{v} = x' \widehat{i'} + y' \widehat{j'} = x \widehat{i} + y \widehat{j}$. We know that $\widehat{i'} = \cos(\phi) \widehat{i} + \sin(\phi) \widehat{j}$ and $\widehat{j'} = \cos(90^\circ + \phi) \widehat{i} + \sin(90^\circ + \phi) \widehat{j} = -\sin(\phi) \widehat{i} + \cos(\phi) \widehat{j}$. We also know that $z = z'$. Now, since the trajectory of the pendulum is defined in the inertial frame, we must introduce a method to transform coordinates from the rotational frame back to the inertial frame. Using our original expression for $\vec{v}$, we can substitute the expressions for $\widehat{i'}$ and $\widehat{j'}$ to find a rotation matrix about the z-axis, $R_{x}(\phi)$, such that

$$
\begin{bmatrix}
x \\
y \\
z
\end{bmatrix} = R_z(\phi)
\begin{bmatrix}
x' \\
y' \\
z'
\end{bmatrix}
$$

Doing so yields:

$$
R_z(\phi) = 
\begin{bmatrix}
\cos \phi & -\sin \phi & 0 \\
\sin \phi & \cos \phi & 0 \\
0 & 0 & 1
\end{bmatrix}
$$

We can now convert the position of the pivot and the mass from the rotating frame to the inertial frame. For the pivot (in the rotating frame):

$$
\vec{r'}_{Pivot} = 
\begin{bmatrix}
w_1 \\
0 \\
h_1
\end{bmatrix}
$$

Therefore, the pivot position in the inertial frame can be expressed as:
$$
\vec{r}_{Pivot} = 
\begin{bmatrix}
w_1 \cos(\Omega t) \\
w_1 \sin(\Omega t) \\
h_1
\end{bmatrix}
$$

The position of the mass the rotating frame relative to the pivot can be expressed as:

$$
\vec{r'}_{Mass, rel} = 
\begin{bmatrix}
L \sin(\theta) \\
0 \\
-L \cos(\theta)
\end{bmatrix}
$$

Where $\theta$ is the angle in the $x'- z'$ plane between the pendulum and the verticle $z'$ axis. Therefore, the position of the mass can be expressed in the inertial frame as:

$$
\vec{r}_{Mass, rel} = 
\begin{bmatrix}
L \sin(\theta) \cos(\Omega t) \\
L \sin(\theta) \sin(\Omega t) \\
-L \cos(\theta)
\end{bmatrix}
$$

The absolute position of the mass in the inertial frame can then be expressed as:

$$
\vec{r}_{Mass} = \vec{r}_{Pivot} + \vec{r}_{Mass, rel} = 
\begin{bmatrix}
\cos(\Omega t)\left[w_1 + L \sin \theta (t)\right] \\
\sin(\Omega t)\left[w_1 + L \sin \theta (t)\right] \\
h_1 - L \cos \theta (t)
\end{bmatrix}
$$

Therefore:

$$
x(t) = \cos(\Omega t)\left[w_1 + L \sin \theta (t)\right]
$$

$$
y(t) = \sin(\Omega t)\left[w_1 + L \sin \theta (t)\right]
$$

$$
z(t) = h_1 - L \cos \theta (t)
$$

Taking the time-derrivative of each allows us to find the speed of the mass, which can then be used to find the kinetic energy of the system:

$$
T = \frac{1}{2} m \left[L^2 \dot{\theta}(t)^2 + \Omega^2 (w_1 + L \sin \theta (t)^2) \right]
$$

The potential energy of the system can also be expressed as:

$$
V = mg \left(h_1 - L \cos \theta (t) \right)
$$

We can now write the Lagrangian as:

$$
L = T - V = \frac{1}{2} m \left[L^2 \dot{\theta}(t)^2 + \Omega^2 (w_1 + L \sin \theta (t)^2) \right] - mg \left(h_1 - L \cos \theta (t) \right)
$$

Finally we can used the Euler-Lagrange equation, $\frac{d}{dt}\left(\frac{\partial L}{\partial \dot{\theta}} \right) - \frac{\partial L}{\partial \theta} = 0 $, to find the equation of motion for the system:

$$
\ddot{\theta}(t) = \frac{\Omega^2}{L} \left(w_1 + L \sin \theta (t) \right) \cos \theta (t) - \frac{g}{L} \sin \theta (t)
$$

**Please note, a full derrivation of the equation above can be found in the project01_team06_notes.pdf file in the notes directory of the repository.** 

Now, in order to program this system in Julia, we must decompose the second-order ODE above into two first-order ODEs. We can do that by defining:

$$
\begin{aligned}
u_1 = \theta \\
u_2 = \dot{\theta}
\end{aligned}
$$

Therefore, we have:

$$
\begin{aligned}
\dot{u}_1 &= u_2 \\
\dot{u}_2 &= \frac{\Omega^2}{L} \left(w_1 + L \sin \theta (t) \right) \cos \theta (t) - \frac{g}{L} \sin \theta (t)
\end{aligned}
$$

Finally, we must determine a low and high speed for $\Omega$. Given that the natural frequency of the pendulum is

$$
\omega_o = \sqrt{\frac{g}{L}} = \sqrt{\frac{9.81}{0.15}} \approx 8.08 \: \text{rad/s}
$$

Therefore, we can choose a low speed of $ \Omega_s = 1 \: \text{rad/s}$ and a high speed of $ \Omega_f = 12 \: \text{rad/s}$. We can also evaluate the cases where the pendulum starts at rest, $\theta_o = 0$, and where the pendulum stars with a small displacement, $\theta_o = 0.05 \: \text{rad}$.
