# Lab 1 — Entropy Mixing

---

# Thermal Physics Lab 1 — Entropy Increase Due to Convection and Conduction

**Physics, Heriot-Watt University**

---

## Overview

In this lab, we will create a simulation designed to illustrate the concept of entropy increase through either convection or conduction. The simulation will show how a system with several hot particles separated from cold particles evolves as the particles mix and exchange energy.  We will start by initialising an array of $N$ particles with the hot particles originally separated from the cold particles.  As time evolves, neighbouring particles can either swap positions, representing convection, or they can exchange energy, representing conduction.  We find that the entropy increases with time, reflecting a loss of spatial information about where the hot particles are.  This is an example of the second law of thermodynamics, which states that the entropy of a closed system will increase or remain constant with time.

This simulation is an example of predictable macroscopic behaviour emerging from simple rules of microscopic states.  We can make some simple assumptions about how two neighbouring particles might interact with each other, and then observe how these influence the properties of the combined system.  

In this case, we the only update rule that we apply is that neighbouring particles can either swap locations (simulating convection) or exchange and average their energies (simulating conduction).  These simple rules then result in a predictable increase in entropy associated with the location of the heat in a system.  An important outcome is that we can be confident of the long-term behaviour of the system (increase in entropy), but we cannot predict what will happen in any one time step (the entropy may go up or down).


---

## Learning Outcomes

By the end of this lab you should be able to:

- Simulate convection and/or conduction of heat via simple update rules.
- Describe entropy as a measure of spatial distribution of energy.
- Understand that small fluctuations can temporarily reduce entropy.
- Explain how microscopic randomness produces predictable macroscopic behaviour.

---

# Background

## Shannon Entropy

The Shannon entropy is

$$
S = -\sum_i p_i \log_2 p_i
$$

where $p_i$ is the probability of a random event occurring.

Using $\log_2$ gives entropy in **bits**.

---

## Gibbs and Boltzmann Entropy

In statistical mechanics,

$$
S = -k_B \sum_i p_i \ln p_i
$$

If all microstates are equally probable, $p_i = 1/\Omega$, giving

$$
S = k_B \ln \Omega.
$$

If $\Omega = 1$, then $S = 0$.

---

## Second Law of Thermodynamics

For an isolated system,

$$
\Delta S_{\text{total}} \ge 0.
$$

Entropy increases until equilibrium is reached.

---

## Convection vs Conduction

- **Convection** → swap neighbouring particles.
- **Conduction** → average neighbouring energies.

---

# Simulation Details

```{figure} ../assets/images/Entropy.png
:width: 90%

Schematic illustration of entropy increase during mixing.
