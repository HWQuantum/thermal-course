---
authors:
  - name: Physics, Heriot-Watt University
exports:
  - format: pdf
    output: ../_build/html/exports/lab3.pdf
downloads:
  - title: Download PDF
    url: /thermal-course/exports/lab3.pdf
---


# Lab 3 - The Heat Diffusion Equation - Fourier Transform
# Overview


In this lab, we will solve the heat diffusion equation via a Fourier-based method, representing the conduction of heat through a solid as time evolves. The simulation will show how a system with an initial localised area of heat will transition to a system where the heat is distributed throughout space.  We will start by initialising an array that represents where the heat in a sample is and then use the Fourier method to solve the equation and update the spatial distribution of heat as time evolves.  We find that the heat distribution at time $t+\Delta t$ is equal to the inverse Fourier transform of the Fourier transform of the current heat distribution multiplied by a Gaussian function.  In the Fourier domain, the multiplication by the Gaussian attenuates the high spatial frequency components of the temperature distribution. 

This simulation is an example of using a Fourier-based approach for solving a second-order partial differential equation.  As with the finite difference method, a similar approach can be used to solve many other similar differential equations. 


---

## Learning Outcomes

By the end of this lab you should be able to:

- Understand the physical principles behind thermal diffusion.
- Learn how to numerically solve the one-dimensional heat equation using Fourier-based methods.
- Implement the method in MATLAB and analyze the temperature evolution in a solid.
- Understand the extension of the method to two and three dimensions.
- Understand the extension of the method to include the addition of heat sources.
- Understand the advantages and disadvantages of this method versus the finite difference approach

---

# Background

## Exponential Decay and Newton's Law of Cooling

Before solving the heat equation, it's helpful to understand a simpler example of exponential decay, which is Newton's Law of Cooling. This describes the temperature \( T(t) \) of an object as it cools in a surrounding environment of constant temperature \( T_{\text{env}} \). The rate of cooling is proportional to the temperature difference
%
\begin{equation}
\frac{dT}{dt} = -\kappa(T - T_{\text{env}}),
\label{eq:newton}
\end{equation}
%
where \( \kappa > 0 \) is a cooling constant.

## Solution

We can solve this first-order ODE by separating variables
%
\[
\frac{dT}{T - T_{\text{env}}} = -\kappa\,dt.
\]
%
Integrating both sides we get
%
\[
\ln|T - T_{\text{env}}| = -\kappa t + C
\quad \Rightarrow \quad
T(t) = T_{\text{env}} + (T_0 - T_{\text{env}}) e^{-\kappa t}.
\]
%
This shows that the temperature approaches the environment temperature exponentially over time.  It is important to note that to find the temperature at any future time, we only need to know the initial conditions ($T_0$ and $T_{\text{env}}$) and the time $t$.

## connection to the Heat Equation

This solution is of the same form as the solution to the Fourier-transformed heat equation (Equation 4 in this document), where each Fourier mode decays exponentially in time:

\[
\hat{T}(k,t) = \hat{T}(k,0) e^{-\alpha k^2 t}
\]

Understanding Newton's Law of Cooling helps recognise exponential decay in thermal processes, and see how the system relaxes toward equilibrium over time.
