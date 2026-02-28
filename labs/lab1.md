---
authors:
  - name: Physics, Heriot-Watt University
exports:
  - format: pdf
    output: ../_build/html/exports/lab1.pdf
downloads:
  - title: Download PDF
    url: /thermal-course/exports/lab1.pdf
---
# Lab 1 - Entropy Increase
# Overview

In this lab, we will create a simulation designed to illustrate the concept of entropy increase through either convection or conduction. The simulation will show how a system with several hot particles separated from cold particles evolves as the particles mix and exchange energy.  We will start by initialising an array of $N$ particles with the hot particles originally separated from the cold particles.  As time evolves, neighbouring particles can either swap positions, representing convection, or they can exchange energy, representing conduction.  We find that the entropy increases with time, reflecting a loss of spatial information about where the hot particles are.  This is an example of the second law of thermodynamics, which states that the entropy of a closed system will increase or remain constant with time.

This simulation is an example of predictable macroscopic behaviour emerging from simple rules of microscopic states.  We can make some simple assumptions about how two neighbouring particles might interact with each other, and then observe how these influence the properties of the combined system.  

In this case, we the only update rule that we apply is that neighbouring particles can either swap locations (simulating convection) or exchange and average their energies (simulating conduction).  These simple rules then result in a predictable increase in entropy associated with the location of the heat in a system.  An important outcome is that we can be confident of the long-term behaviour of the system (increase in entropy), but we cannot predict what will happen in any one time step (the entropy may go up or down).

---

## How will you be assessed?

At the end of the lab, you will hand in a single MATLAB .m file that contains your simulation and generates your results.  You should comment your code throughout, so that it is clear that you understand the physics and the results.  Make sure that all plots and graphs have suitable axes and labels.

In order to pass this lab, your code must run and generate results in line with the learning objectives below. This is a pass/fail lab, so you will be given 100% if your code meets the minimum requirements, and 0% if you do not.

**In addition to the baseline results, you are expected to try some of the “Further Investigation," detailed at the end.**

---

## Learning Outcomes

The objectives of this lab are to:

- How to simulate convection and/or conduction of heat via simple update rules.
- Describe entropy as a measure of the spatial distribution of energy.
- Understand that small fluctuations can lead to a decrease in entropy, but the second law governs the global behaviour as time increases.
- Explore how microscopic random motion relates to macroscopic thermodynamic behaviour.

---

# Background

## Shannon entropy

The Shannon entropy $S$ is given by

$$
S = -\sum_{i} p_i \log_2 p_i,
$$

where $p_i$ is the probability of a random event occurring.  The Shannon entropy is dimensionless as it is based on probabilities, and the units depend on the base of the logarithm.  In this case, we are using $\log_2$, so the units of $S$ are bits.

<div id="coin-entropy-widget" style="border:1px solid #e5e7eb;border-radius:12px;padding:16px;max-width:980px;">
  <div style="display:flex;gap:16px;flex-wrap:wrap;align-items:center;justify-content:space-between;">
    <div style="font-weight:600;font-size:18px;">Coin bias → Shannon entropy</div>
    <div id="summary" style="font-size:16px;font-weight:600;"></div>
  </div>

  <div style="margin-top:10px;display:flex;gap:14px;align-items:center;flex-wrap:wrap;">
    <label for="pSlider" style="min-width:130px;font-weight:600;">p(Heads)</label>
    <input id="pSlider" type="range" min="0" max="1" step="0.01" value="0.50" style="flex:1;min-width:260px;">
    <div id="pVal" style="width:64px;text-align:right;font-variant-numeric:tabular-nums;">0.50</div>
  </div>

  <div style="margin-top:14px;display:grid;grid-template-columns:1fr 1fr;gap:14px;">
    <div style="border:1px solid #e5e7eb;border-radius:10px;padding:10px;">
      <div style="font-weight:600;margin-bottom:6px;">Entropy H(p) [bits]</div>
      <svg id="plotH" viewBox="0 0 460 260" width="100%" height="260" aria-label="Entropy plot"></svg>
    </div>
    <div style="border:1px solid #e5e7eb;border-radius:10px;padding:10px;">
      <div style="font-weight:600;margin-bottom:6px;">Effective modes 2^H(p)</div>
      <svg id="plotN" viewBox="0 0 460 260" width="100%" height="260" aria-label="Modes plot"></svg>
    </div>
    <div style="grid-column:1 / -1;border:1px solid #e5e7eb;border-radius:10px;padding:10px;">
      <div style="font-weight:600;margin-bottom:6px;">Outcome probabilities</div>
      <svg id="plotBar" viewBox="0 0 940 260" width="100%" height="260" aria-label="Bar chart"></svg>
    </div>
  </div>

  <div style="margin-top:10px;color:#6b7280;font-size:14px;">
    Tip: maximum entropy occurs at p = 0.5 (fair coin). At p = 0 or 1 the outcome is fully predictable.
  </div>
</div>

<script>
(() => {
  const el = document.getElementById("coin-entropy-widget");
  if (!el) return;

  const slider = el.querySelector("#pSlider");
  const pVal = el.querySelector("#pVal");
  const summary = el.querySelector("#summary");
  const plotH = el.querySelector("#plotH");
  const plotN = el.querySelector("#plotN");
  const plotBar = el.querySelector("#plotBar");

  const clamp = (x, a, b) => Math.max(a, Math.min(b, x));
  const Hbits = (p) => {
    const eps = 1e-12;
    p = clamp(p, eps, 1 - eps);
    const q = 1 - p;
    return -(p * Math.log2(p) + q * Math.log2(q));
  };

  // Simple SVG helpers
  const clear = (svg) => { while (svg.firstChild) svg.removeChild(svg.firstChild); };
  const line = (svg, x1,y1,x2,y2, stroke="#111827", w=1) => {
    const e = document.createElementNS("http://www.w3.org/2000/svg","line");
    e.setAttribute("x1",x1); e.setAttribute("y1",y1);
    e.setAttribute("x2",x2); e.setAttribute("y2",y2);
    e.setAttribute("stroke",stroke); e.setAttribute("stroke-width",w);
    svg.appendChild(e); return e;
  };
  const path = (svg, d, stroke="#111827", w=2, fill="none") => {
    const e = document.createElementNS("http://www.w3.org/2000/svg","path");
    e.setAttribute("d", d);
    e.setAttribute("stroke", stroke);
    e.setAttribute("stroke-width", w);
    e.setAttribute("fill", fill);
    e.setAttribute("stroke-linejoin","round");
    e.setAttribute("stroke-linecap","round");
    svg.appendChild(e); return e;
  };
  const circle = (svg, cx,cy,r, fill="#111827") => {
    const e = document.createElementNS("http://www.w3.org/2000/svg","circle");
    e.setAttribute("cx",cx); e.setAttribute("cy",cy); e.setAttribute("r",r);
    e.setAttribute("fill",fill);
    svg.appendChild(e); return e;
  };
  const text = (svg, x,y, str, size=12, anchor="start", fill="#111827") => {
    const e = document.createElementNS("http://www.w3.org/2000/svg","text");
    e.setAttribute("x",x); e.setAttribute("y",y);
    e.setAttribute("fill",fill);
    e.setAttribute("font-size",size);
    e.setAttribute("font-family","system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif");
    e.setAttribute("text-anchor",anchor);
    e.textContent = str;
    svg.appendChild(e); return e;
  };
  const rect = (svg, x,y,w,h, fill="#111827") => {
    const e = document.createElementNS("http://www.w3.org/2000/svg","rect");
    e.setAttribute("x",x); e.setAttribute("y",y);
    e.setAttribute("width",w); e.setAttribute("height",h);
    e.setAttribute("fill",fill);
    svg.appendChild(e); return e;
  };

  function drawAxes(svg, W, H, padL, padR, padT, padB, xLabel, yLabel, yTicks) {
    // Frame
    const x0 = padL, y0 = H - padB, x1 = W - padR, y1 = padT;
    // axes
    line(svg, x0, y0, x1, y0, "#111827", 1);
    line(svg, x0, y0, x0, y1, "#111827", 1);

    // x ticks (0..1)
    for (let i=0;i<=5;i++){
      const t=i/5, x = x0 + t*(x1-x0);
      line(svg,x,y0,x,y0+4,"#111827",1);
      text(svg,x,y0+18,(t).toFixed(1),11,"middle","#374151");
    }
    text(svg,(x0+x1)/2, H-6, xLabel, 12, "middle", "#111827");

    // y ticks
    yTicks.forEach(({v,label})=>{
      const y = y0 - v*(y0-y1);
      line(svg, x0-4, y, x0, y, "#111827", 1);
      text(svg, x0-8, y+4, label, 11, "end", "#374151");
      // faint grid
      line(svg, x0, y, x1, y, "#e5e7eb", 1);
    });

    // y label (simple)
    text(svg, 12, (y0+y1)/2, yLabel, 12, "middle", "#111827")
      .setAttribute("transform", `rotate(-90 12 ${(y0+y1)/2})`);
    return {x0,y0,x1,y1};
  }

  function drawCurve(svg, box, fY, yMax) {
    const {x0,y0,x1,y1} = box;
    const n=400;
    let d="";
    for (let i=0;i<=n;i++){
      const p=i/n;
      const y=fY(p);
      const xPix = x0 + p*(x1-x0);
      const yPix = y0 - (y/yMax)*(y0-y1);
      d += (i===0 ? "M":"L") + xPix.toFixed(2) + " " + yPix.toFixed(2) + " ";
    }
    path(svg, d, "#1f2937", 2);
  }

  function drawPoint(svg, box, p, y, yMax) {
    const {x0,y0,x1,y1} = box;
    const xPix = x0 + p*(x1-x0);
    const yPix = y0 - (y/yMax)*(y0-y1);
    circle(svg, xPix, yPix, 5, "#111827");
  }

  function render(p) {
    p = clamp(p, 0, 1);
    const q = 1 - p;
    const H = Hbits(p);
    const Neff = Math.pow(2, H);

    pVal.textContent = p.toFixed(2);
    summary.textContent = `H = ${H.toFixed(3)} bits, 2^H = ${Neff.toFixed(3)}`;

    // Plot H
    clear(plotH);
    const WH = 460, HH = 260;
    const boxH = drawAxes(
      plotH, WH, HH, 52, 18, 16, 40,
      "p (heads)", "H(p)", [
        {v:0.0,label:"0.0"},
        {v:0.25,label:"0.25"},
        {v:0.50,label:"0.50"},
        {v:0.75,label:"0.75"},
        {v:1.00,label:"1.0"}
      ]
    );
    drawCurve(plotH, boxH, (x)=>Hbits(x), 1.0);
    drawPoint(plotH, boxH, p, H, 1.0);

    // Plot Neff
    clear(plotN);
    const boxN = drawAxes(
      plotN, WH, HH, 64, 18, 16, 40,
      "p (heads)", "2^H", [
        {v:(1.0-0.9)/(2.1-0.9), label:"1.0"},
        {v:(1.2-0.9)/(2.1-0.9), label:"1.2"},
        {v:(1.5-0.9)/(2.1-0.9), label:"1.5"},
        {v:(1.8-0.9)/(2.1-0.9), label:"1.8"},
        {v:(2.1-0.9)/(2.1-0.9), label:"2.1"}
      ]
    );
    // custom mapping for this axis: y in [0.9,2.1]
    const yMin=0.9, yMax=2.1;
    const {x0,y0,x1,y1} = boxN;
    // draw curve
    let d="";
    const n=400;
    for (let i=0;i<=n;i++){
      const pp=i/n;
      const yy=Math.pow(2, Hbits(pp));
      const xPix=x0 + pp*(x1-x0);
      const yPix=y0 - ((yy-yMin)/(yMax-yMin))*(y0-y1);
      d += (i===0?"M":"L") + xPix.toFixed(2) + " " + yPix.toFixed(2) + " ";
    }
    path(plotN, d, "#1f2937", 2);
    // point
    const xPix=x0 + p*(x1-x0);
    const yPix=y0 - ((Neff-yMin)/(yMax-yMin))*(y0-y1);
    circle(plotN, xPix, yPix, 5, "#111827");

    // Bar chart
    clear(plotBar);
    const WB=940, HB=260;
    // axes
    const padL=70, padR=18, padT=16, padB=42;
    const bx0=padL, by0=HB-padB, bx1=WB-padR, by1=padT;
    line(plotBar, bx0, by0, bx1, by0, "#111827", 1);
    line(plotBar, bx0, by0, bx0, by1, "#111827", 1);
    // y grid/ticks 0..1
    for (let i=0;i<=5;i++){
      const t=i/5;
      const y=by0 - t*(by0-by1);
      line(plotBar, bx0-4, y, bx0, y, "#111827", 1);
      text(plotBar, bx0-8, y+4, t.toFixed(1), 11, "end", "#374151");
      line(plotBar, bx0, y, bx1, y, "#e5e7eb", 1);
    }
    text(plotBar, 18, (by0+by1)/2, "Probability", 12, "middle", "#111827")
      .setAttribute("transform", `rotate(-90 18 ${(by0+by1)/2})`);

    const barW=180;
    const gap=180;
    const xHeads = bx0 + 180;
    const xTails = xHeads + barW + gap;
    const hHeads = p*(by0-by1);
    const hTails = q*(by0-by1);

    rect(plotBar, xHeads, by0-hHeads, barW, hHeads, "#4b5563");
    rect(plotBar, xTails, by0-hTails, barW, hTails, "#9ca3af");
    text(plotBar, xHeads + barW/2, by0+20, "Heads", 12, "middle", "#111827");
    text(plotBar, xTails + barW/2, by0+20, "Tails", 12, "middle", "#111827");
  }

  slider.addEventListener("input", () => render(parseFloat(slider.value)));
  render(parseFloat(slider.value));
})();
</script>
---

## Gibbs and Boltzmann's entropy

In statistical mechanics, the Gibbs entropy is defined as

$$
S = -k_B \sum_{i} p_i \ln p_i
$$

where $p_i$ is the probability of the system being in microstate $i$, and $k_B$ is Boltzmann's constant.  The Gibbs entropy has units of Joules per Kelvin $J K^{-1}$, which comes from Boltzmann's constant $k_B$.

If all microstates are equally probable, then we have $p_i = 1 / \Omega$, where $\Omega$ is the total number of states.  If we insert this probability in the Gibbs entropy, we obtain the common Boltzmann form of entropy

$$
S = -k_B \sum_{i} \frac{1}{\Omega} \ln\frac{1}{\Omega} = k_B \ln \Omega.
$$

We see that if there is only one possible microstate, then the entropy is given by $S=k_B\ln 1=0~J K^{-1}$, as is equal to zero. 

---

## Second law of thermodynamics

In any isolated system, the total entropy can never decrease over time. It either increases or stays the same, reaching a maximum at equilibrium.  Entropy is a measure of how many microstates correspond to a given macrostate (how disordered or probable the system is).  The second law states that

$$
\Delta S_{\text{total}} \ge 0,
$$

where $\Delta S_{\text{total}}$ is the change in entropy of the entire system.  

---

## Convection and conduction

Convection is the transfer of heat by the movement of molecules within a liquid or gas.  Hot regions become of liquid or gas have a low density, while cooler regions are denser.  This results in a flow of particles that moves heat. We will simulate convection by swapping the location of two adjacent particles. 

Conduction is the transfer of heat through a material without the movement of the material itself.  This happens when faster-moving (hotter) particles collide with slower ones, passing on energy.  We will simulate conduction by the averaging of the heat energies of two adjacent particles.

---

# Simulation details


We start by initialising the system, then letting the system evolve according to the physics rules we have established (convection or conduction).  We then calculate properties associated with the system, e.g. the Gibbs entropy associated with the temperature distribution, and then plot and analyse the results.  Strictly, the Gibbs entropy uses the microstates of a system, but it is often not possible to know or count all of the microstates of a system.  In such a case, we can use a coarse-grained version where we calculate the fraction of total energy (or hot particle content) in spatial region $i$.

To achieve this, we will divide the total system into several (2 or 4) regions, and calculate the probability of finding a hot particle in each region.  We always require a probability distribution to calculate entropy, and we will use the probability distribution of heat in this simulation in the Gibbs formulation.  The heat probability distribution provides the necessary information about “where the heat is in the system,” which is what is needed to calculate the entropy.

---

# Physical Interpretation

```{figure} ../assets/images/Entropy.png
:width: 80%
:align: center
:name: fig-entropy

Schematic illustration of the simulation.  We start with an initial distribution of cold particles on the left and hot particles on the right.  The probability distribution measures where we find the hot particles.  At $t = 0$, these are all on the right-hand side, and this corresponds to an entropy of 0 bits.  As $t$ increases, we update the distribution and calculate $S_t$.   After many iterations, the entropy will tend to the maximum value, $S_{t>10^6} \approx k_B \ln 2 = 0.69~k_B$.
```

For a system with distinguishable regions and energy distribution, entropy can be related to our ability to localize energy (e.g., hot particles). The entropy measures the distribution of this energy throughout the system

At the start of this simulation, all hot particles are on the right and all cold particles on the left, therefore, the system is highly ordered and low in entropy. As particles move and mix, it becomes increasingly difficult to determine where the hot particles are, indicating an increase in entropy. As particles swap places and energy becomes more uniformly distributed, the number of possible microstates increases, and so too does the entropy.

This is exactly what is contained in the second law of thermodynamics, which states that in an isolated system, entropy tends to increase with time. In our simulation, there is no energy input or output, and yet the system evolves from a low-entropy state (localized energy) to a high-entropy state (spread-out energy). The entropy increase is a direct result of this spatial delocalization.

# Pseudo Code

```{admonition} Algorithm
:class: algorithm
:name: alg-entropy-mixing

**Entropy Mixing Simulation (1D)**

**Initialisation**
1. Initialise a vector $v$ of length $N$ (e.g 100) with values 0 on the left for cold and 1 on the right for hot.
2. Set the total number of time steps $T$ (e.g. 10,000), set the number of bins $B$ for entropy calculation (e.g. 2, 4, or 8), and initialise a vector of length $T$ to store entropy values $S$.  Below, we have $B=2$.

**For each time step $t$ from 1 to $T$:**
1. Randomly choose an index $i$ in $1,\dots,N$
2. If $i = 1$: set neighbour index $j = 2$  
   Else if $i = N$: set neighbour index $j = N-1$  
   Else: randomly choose $j = i - 1$ or $j = i + 1$
3. Swap the values to simulate convection ($v[i] \leftrightarrow v[j]$)
4. Compute the probability distribution of heat $P$
5. Divide $v$ into two equal spatial bins $b= \{1,2\}$
6. For each bin $b$:  
   $p_b = (\text{sum of values in bin } b) / (\text{sum of all values})$
7. Compute entropy at time step $t$:  
   $$ S_t = -k_B \sum_{b=1}^{2} p_b \ln p_b $$
8. Store $S_t$ in entropy array
9. Plot $v_t$, $P$, and $S_t$ every 100 iterations
```

# Target Results

```{figure} ../assets/images/Final_result.png
:width: 80%
:align: center
:name: target

Target results.
```

# Further investigation

Here are some suggestions for further development of the simulation:
- Different initial conditions: Try starting with alternating hot and cold particles or a Gaussian heat profile.
- Simulation of conduction: Replace the swapping of the particles at each iteration with averaging.  This would represent conduction rather than convection.
- Track mean hot particle position: Track and plot the mean position of hot particles over time.
- 2D extension: Extend the system to a 2D lattice and observe entropy changes as particles diffuse in two dimensions.