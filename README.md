## OutcomeForge: Advanced Prediction Market Protocol on Aptos

**OutcomeForge** implements Paradigm’s **Prediction Market Automated Market Maker (PM‑AMM)** on the Aptos blockchain. It translates rigorous financial theory into modular Move smart contracts, leveraging the Aptos Fungible Asset framework to deliver a capital‑efficient, mathematically precise, and decentralized prediction market protocol.

---

## 1. Overview

### 1.1 Introduction
Prediction markets rely on **outcome tokens** with binary payoffs:
- Resolve to **1** if the event occurs.
- Resolve to **0** if the event does not occur.

Traditional AMMs (e.g., Constant Product $$\(x \cdot y = k\))$$ fail for these instruments because:
1. **Guaranteed LP loss at expiry** — arbitrage drains valuable tokens, leaving worthless ones.
2. **Inefficient liquidity profile** — uniform distribution across infinite ranges mismatched to bounded $$\([0,1]\)$$ outcome token prices.

---

### 1.2 Foundational Concepts

#### Gaussian Score Dynamics
Outcome token prices are modeled as probabilities derived from a Gaussian process (Brownian motion). The price is the probability that the underlying score exceeds a threshold at expiration:



$$\[
P = \Phi\left(\frac{y - x}{L}\right)
\]$$



Where:
- $$\(x\)$$ = reserve of NO tokens
- $$\(y\)$$ = reserve of YES tokens
- $$\(L\)$$ = liquidity parameter
- $$\(\Phi(z)\)$$ = CDF of the standard normal distribution

This ensures prices are valid probabilities between 0 and 1.

#### Loss vs. Rebalancing (LVR)
LVR quantifies expected LP losses due to arbitrage against stale prices. Unlike impermanent loss, it isolates the unavoidable cost of passive market‑making. LPs can model profitability by comparing expected LVR against fee revenue.

---

### 1.3 Static pm‑AMM

The invariant curve is defined as:



$$\[
(y - x)\Phi\left(\frac{y - x}{L}\right) + L\phi\left(\frac{y - x}{L}\right) - y = 0
\]$$



Where:
- $$\(\phi(z) = \frac{1}{\sqrt{2\pi}} e^{-z^2/2}\)$$ (PDF of normal distribution)
- $$\(\Phi(z)\)$$ = CDF of normal distribution

**Economic intuition:**  
Liquidity is concentrated around $$\(p = 0.5\)$$, withdrawn from extremes (0 or 1) where LP risk is highest. This mirrors real‑world trading activity, which is densest when uncertainty is greatest.

---

### 1.4 Dynamic pm‑AMM

Extends the static model to account for **time decay** and volatility near expiration.

Dynamic invariant:



$$\[
(y - x)\Phi\left(\frac{y - x}{L\sqrt{T - t}}\right) + L\sqrt{T - t}\,\phi\left(\frac{y - x}{L\sqrt{T - t}} \right) - y = 0
\]$$



Where:
- $$\(T\)$$ = expiration time
- $$\(t\)$$ = current time

Liquidity decays as $$\(L\sqrt{T - t}\)$$, reducing LP exposure when risk is highest.

#### Constant Expected LVR
Instantaneous LVR:



$$\[
\mathsf{LVR}_t = \frac{V_t}{2(T - t)}
\]$$



Expected constant rate:



$$\[
\mathbb{E} = \frac{V_0}{2T}
\]$$



Total expected LVR over lifetime:



$$\[
\int_0^T \frac{V_0}{2T}\,dt = \frac{V_0}{2}
\]$$



**Result:** LPs lose exactly half their initial capital to arbitrage over the market’s lifetime — a predictable, quantifiable cost.

---

### 1.5 Model Comparison

| Feature               | CPMM $$(\(x \cdot y = k\))$$ | LMSR | Static pm‑AMM | Dynamic pm‑AMM |
|------------------------|--------------------------|------|---------------|----------------|
| Asset suitability      | General tokens           | Subsidized PMs | Outcome tokens | Outcome tokens |
| Liquidity profile      | Uniform infinite range   | Subsidy‑driven | Concentrated at $$\(p=0.5\)$$ | Time‑decaying |
| LVR behavior           | High/unpredictable       | High at extremes | Uniform across price | Uniform across price & time |
| LP loss profile        | Guaranteed loss at expiry | Subsidized | Predictable vs. price | Predictable vs. lifetime |

---

## 2. Architecture

Morpheus separates **mathematical logic** from **stateful contract management** for auditability and robustness.

- `invariant.move`: Implements PM‑AMM invariant equations.
- `swap_math.move`: Pure swap calculations.
- `liquidity_math.move`: Value‑based liquidity addition/removal.
- `prediction_market.move`: Market lifecycle, collateral, FA tokens.
- `pool_state.move`: Reserves, liquidity parameter $$\(L\)$$, fees, dynamic decay.
- `normal_dist.move`: Implements $$\(\phi\)$$, $$\(\Phi\)$$, and inverse CDF.

---

## 3. Core User Actions

- **Swap:** YES ↔ NO tokens, priced via $$\(\Phi\)$$.
- **Mint Pair:** Deposit collateral → receive YES + NO tokens.
- **Add Liquidity:** Specify desired value increase $$\(\Delta V(P)\)$$.
  - Formula:
    

$$\[
    \Delta L = \frac{\Delta V(P)}{\phi(\Phi^{-1}(P))}
    \]$$


- **Remove Liquidity:** Burn LP tokens → proportional reserves + fees.
- **Resolve Market:** Creator declares outcome (YES/NO).
- **Settle Tokens:** Winning tokens redeemable for collateral.

---

## 4. Worked Example

Suppose:
- $$\(x = 500\), \(y = 500\), \(L = 1000\)$$.  
- Price of YES token:
  

$$\[
  P = \Phi\left(\frac{500 - 500}{1000}\right) = \Phi(0) = 0.5
  \]$$


- Liquidity addition: desired $$\(\Delta V(P) = 50\)$$.  
  

$$\[
  \Delta L = \frac{50}{\phi(\Phi^{-1}(0.5))} = \frac{50}{\phi(0)} = \frac{50}{1/\sqrt{2\pi}} \approx 125.3
  \]$$



Thus, the LP must deposit tokens corresponding to $$\(\Delta L \approx 125.3\)$$ to increase pool value by 50 at price 0.5.

---

## 5. Conclusion

Morpheus advances prediction market design by:
- Modeling outcome token dynamics via Gaussian processes.
- Providing **uniform, predictable LP risk** across price and time.
- Delivering a **capital‑efficient, decentralized AMM** specialized for binary outcome tokens.

This represents a shift from heuristic AMMs to **explicit, model‑based risk management**, setting a foundation for the next generation of DeFi protocols.
