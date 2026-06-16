# Gas Safety Calculator Methodology

This document outlines the theoretical models, assumptions, and potential limitations of the University of Nottingham Gas/Cryogen Safety Calculator. It serves as a technical justification for the calculation approach.

## 1. Core Mathematical Models

The calculator assesses the hazard of a gas release by determining the resulting atmospheric concentration. This is evaluated in two distinct scenarios: an **Instantaneous Release** (worst-case total loss of contents) and a **Continuous Slow Leak**.

### 1.1 Volume Calculations
Before concentration can be determined, the total volume of gas (at standard atmospheric pressure and temperature) must be calculated from the source container.

*   **Compressed Gas Volume:** 
    `Volume (m³) = (Pressure (bar) × (Cylinder Water Capacity (L) / 1000)) / 1.013`
    *   *Theory:* Approximated using Boyle's Law ($P_1V_1 = P_2V_2$), adjusted for standard atmospheric pressure (1.013 bar). 
*   **Liquefied Gas Volume:**
    `Volume (m³) = Weight of Contents (kg) × Specific Volume (m³/kg)`
    *   *Theory:* Direct mass-to-volume conversion using the specific volume of the gas at standard conditions.
*   **Cryogenic Gas Volume:**
    `Volume (m³) = (Liquid Volume (L) × Expansion Ratio) / 1000`
    *   *Theory:* Utilizes standard liquid-to-gas expansion ratios (e.g., 1:694 for Liquid Nitrogen, 1:757 for Liquid Helium, 1:535 for Liquid Carbon Dioxide).

### 1.2 Room Concentration (Instantaneous Release)
Models a catastrophic failure where the entire container contents are released immediately.

*   **General/Flammable/CO₂ Concentration (%):**
    `Concentration (%) = (Gas Volume (m³) / Room Volume (m³)) × 100`
*   **Toxic Gas Concentration (ppm):**
    `Concentration (ppm) = (Gas Volume (m³) / Room Volume (m³)) × 1,000,000`
*   **Asphyxiant Concentration (O₂%):**
    `O₂% = ((Room Volume (m³) - Gas Volume (m³)) × 0.2095 / Room Volume (m³)) × 100`
    *   *Theory:* Assumes normal air contains 20.95% oxygen. The released gas displaces an equal volume of room air. The remaining oxygen is recalculated based on the new total volume of "normal air" left in the space.

### 1.3 Continuous Slow Leak
Models a steady leak into a room with active forced ventilation.

*   **Ventilation Rate ($Q$):**
    `Q (m³/hr) = Room Volume (m³) × Air Changes Per Hour (ACH)`
*   **Steady State Concentration (%):**
    `Concentration (%) = (Leak Rate (m³/hr) / Q) × 100`
    *   *Theory:* Based on the simplified steady-state mass balance equation where the rate of gas entering the room equals the rate of gas being extracted by ventilation.

#### 1.3.1 Advanced Leak-Rate Estimator (Derivations)
An optional helper is provided alongside the **Slow Leak** input to generate a rough leak-rate estimate where the user does not already know the leak rate. These formulae are based on established academic safety protocols (e.g., University of Oxford M1/14) derived from fundamental fluid dynamics.

*   **Gas Line (Post-Regulator):**
    `Q_gas (m³/hr) = 0.4 × P_abs × D² × F_g`
    *   **Derivation:** This is a simplified form of the **Choked Flow Equation** for nitrogen/air at room temperature ($20^\circ C$). When the upstream pressure exceeds ~1.9 bar absolute, the gas velocity at the leak point becomes sonic (choked).
    *   The mass flow rate ($\dot{m}$) is given by: $\dot{m} = C_d A P_1 \sqrt{\frac{\gamma M}{R T_1} (\frac{2}{\gamma+1})^{\frac{\gamma+1}{\gamma-1}}}$
    *   For Nitrogen ($M=0.028$, $\gamma=1.4$) at $20^\circ C$, and assuming a discharge coefficient ($C_d$) of approximately 0.85-0.9 to account for typical sharp-edged leak geometries, the conversion to volumetric flow in $m^3/hr$ with $P$ in bar and $D$ in mm results in a coefficient of $\approx 0.4$. This constant provides a conservative, screening-level estimate for post-regulator leaks.
    *   *Inputs:* Post-regulator line pressure in bar (converted internally to absolute pressure by adding 1 bar), equivalent hole diameter in mm, and a gas factor ($F_g$) relative to Nitrogen.

*   **Liquid Nitrogen Line:**
    `Q_LN2,gas (m³/hr) = 19.3 × D² × √P_gauge`
    *   **Derivation:** This models the gaseous nitrogen hazard resulting from a **liquid** nitrogen leak that subsequently flashes to gas. It combines **Bernoulli's Principle** for liquid flow with the **Liquid-to-Gas Expansion Ratio**.
    *   1. **Liquid Flow:** $v = C_d \sqrt{2 \Delta P / \rho_L}$ (where $\rho_L \approx 808 kg/m^3$).
    *   2. **Expansion:** Nitrogen expands **694 times** by volume when transitioning from liquid to gas at standard conditions.
    *   3. **Integration:** Converting liquid velocity ($v$) through an area ($A = \pi D^2/4$) into a hourly gas volume ($Q$) with $D$ in mm and $P$ in bar, and using a standard $C_d \approx 0.62$ for liquid orifices, yields: $Q \approx 3600 \times 0.62 \times (\frac{\pi (D/1000)^2}{4}) \times \sqrt{\frac{2 \times P \times 10^5}{808}} \times 694 \approx 19.3 \times D^2 \sqrt{P}$.
    *   *Inputs:* Gauge pressure in bar and equivalent hole diameter in mm.
    *   *Note:* This calculates the equivalent **gaseous nitrogen hazard** for ODH screening, not the liquid volume flow.

*   **Liquid Carbon Dioxide Line:**
    `Q_LCO2,gas (m³/hr) = 13.1 × D² × √P_gauge`
    *   **Derivation:** Same approach as the Liquid Nitrogen Line, substituting liquid CO₂ parameters.
    *   1. **Liquid Flow:** $v = C_d \sqrt{2 \Delta P / \rho_L}$ (where $\rho_L \approx 1032 kg/m^3$ for LCO₂ at typical storage conditions of ~20 bar, −20°C).
    *   2. **Expansion:** CO₂ expands **535 times** by volume when transitioning from liquid to gas at standard conditions.
    *   3. **Integration:** Combining as with LN₂: $Q \approx 3600 \times 0.62 \times (\frac{\pi (D/1000)^2}{4}) \times \sqrt{\frac{2 \times P \times 10^5}{1032}} \times 535 \approx 13.1 \times D^2 \sqrt{P}$.
    *   *Inputs:* Gauge pressure in bar and equivalent hole diameter in mm.
    *   *Note:* This formula assumes complete, instantaneous gasification and therefore **overestimates** the immediate release rate (conservative for screening). In practice, some LCO₂ may form dry ice snow that sublimates gradually. Cold CO₂ gas is dense and will pool at floor level - the room-average concentration from the Slow Leak calculation may underestimate local floor-level hazard.

*   **Important Limitation:** These formulae are included only to support the calculator's existing screening approach. They must not be treated as design equations, and they are not suitable for relief devices, long narrow tubing, vessel rupture, pool formation, vacuum-jacket failure, measured leakage verification, or other detailed engineering calculations.

### 1.4 Worst-Case Time to Hazard (0 ACH)
Models the time required to reach a dangerous threshold if the building ventilation completely fails, creating a sealed room.

*   **Time to Target Fraction:**
    `Time (hrs) = - (Room Volume (m³) / Leak Rate (m³/hr)) × ln(1 - Target Fraction)`
    *   *Theory:* Derived from the general accumulation formula $C(t) = \frac{G}{Q}(1 - e^{-\frac{Q}{V}t})$. When $Q$ (ventilation) approaches 0, the equation simplifies to a linear or logarithmic accumulation depending on whether the displacing gas forces air out of the room (used here for O₂ and CO₂).

### 1.5 Personal Bubble Assessment
A secondary calculation applied to all scenarios to estimate the localised concentration immediately surrounding the leak strictly within a 12 m³ zone (representing a 2×2×3m workspace).

*   *Theory:* For instantaneous releases, it calculates the concentration if the gas only expanded to fill the 12 m³ volume. For slow leaks, it applies the room's ACH to the 12 m³ volume to find a localised steady state.

***

## 2. Key Assumptions Built into the Calculator

The calculator relies heavily on idealised scenarios to provide a functional, conservative estimate for risk assessment purposes.

1.  **Perfect Mixing (Uniform Dispersion)**
    The core assumption is that the released gas instantly and perfectly mixes with the air in the specified volume (either the whole room or the Personal Bubble). 
2.  **Ideal Gas Behaviour**
    The calculations for compressed gas volume ignore the compressibility factor ($Z$) of real gases at high pressures. While Boyle's Law provides a close estimate, real gases deviate from ideal behaviour, especially at >200 bar.
3.  **Complete Displacement (Asphyxiants)**
    The asphyxiant model assumes the released gas forces an equivalent volume of perfectly mixed "room air" out of the space through gaps/vents, rather than displacing pure oxygen or creating a pressurised vessel.
4.  **Isothermal Conditions**
    The calculations assume the gas release does not drastically drop the room temperature. In reality, large releases (especially cryogens) will significantly lower the temperature, increasing the density of the cold gas and causing it to slump to the floor.
5.  **Room Free Volume**
    The calculator offers a simple "40% reduction" toggle for cluttered rooms. This is an arbitrary (though conservatively safe) gross estimate of the space occupied by benches, equipment, and cabinets.

***

## 3. Potential Flaws and Limitations of the Approach

When defending this calculator, it is crucial to acknowledge its limitations. It is designed as a **screening tool for worst-case scenarios**, not a computational fluid dynamics (CFD) simulator.

### Limitation 1: Ignoring Gas Stratification & Buoyancy
*   **The Flaw:** By assuming perfect mixing, the calculator severely underestimates local hazards from heavy or light gases. 
*   **Reality:** Cold gases (cryogens), heavy gases (CO₂, Argon), and liquefied petroleum gases will pool on the floor. Light gases (Hydrogen, Helium) will rapidly rise to the ceiling. 
*   **Impact:** If 1m³ of CO₂ is released into a 10m³ room, the calculator shows a 10% average concentration. In reality, the concentration at floor level might be 100%, while at ceiling height it might be 0%. **The calculator may report a "safe" average when a lethal pocket exists.**

### Limitation 2: The "Personal Bubble" is a Rigid Construct
*   **The Flaw:** The 12 m³ bubble provides a useful proximity warning, but the fixed volume is arbitrary.
*   **Reality:** A high-pressure jet release might shoot past a person, while a slow, heavy leak might pool silently around their feet. 
*   **Impact:** The bubble calculation attempts to bridge the gap between "perfect mixing" and "stratification" by forcing a smaller arbitrary mixing zone, but it does not account for the physical mechanics of the specific release.

### Limitation 3: High-Pressure Non-Ideality
*   **The Flaw:** Using Boyle's law without the $Z$-factor for compressed cylinders at 200-300 bar slightly underestimates the total volume.
*   **Reality:** Gases like Nitrogen or Argon are less compressible at high pressures. A 50L cylinder at 230 bar actually holds slightly more gas than $(230 \times 50) / 1.013$ would suggest.
*   **Impact:** For general risk screening, this error is usually acceptable (often less than 5-10%), but it means the volume calculation is not strictly chemically accurate for high-pressure industrial cylinders.

### Limitation 4: Ventilation Short-Circuiting
*   **The Flaw:** In the slow leak calculation, the formula $Q = V \times ACH$ assumes the ventilation perfectly clears the mixed air.
*   **Reality:** In many labs, fresh air from a ceiling supply may flow directly into a ceiling extract without properly turning over the air at floor level (short-circuiting).
*   **Impact:** The steady-state concentration calculation could underestimate the risk if the ventilation is ineffective in the specific area where the leak occurs.

### Conclusion for Defence
The calculator is highly defensible **provided its scope is clearly defined**. It is a conservative, first-pass screening tool designed to trigger the implementation of further controls (like fixed detection or mechanical ventilation) by modelling idealised worst-case scenarios. It cannot-and does not attempt to-replace a competent person considering the specific physics, geometry, and ventilation dynamics of a laboratory space.
