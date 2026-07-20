# Asian Option Pricing & Greeks

Numerical methods for pricing **European and American arithmetic Asian options** with full Greeks computation. Implements Crank-Nicolson finite differences, Monte Carlo with variance reduction, Longstaff-Schwartz for American exercise, and supporting analytics.

**Karan Parihar — MQF Candidate, Rutgers University**

---

## What's in This Repo

| File | Description |
|------|-------------|
| `asian_options_extended.ipynb` | Main notebook — all 8 sections |
| `report/ProjectReport_KaranParihar.pdf` | Original January 2026 project report |
| `report/ExtendedReport_KaranParihar.docx` | Full extended report (this version) |
| `requirements.txt` | Python dependencies |

---

## Notebook Sections

### Original Work (January 2026)

**0A · European Floating-Strike CN (1D)**  
Vécer dimension reduction maps V(t,S,A) = S·H(t,R) with R = A/S, yielding a 1D PDE solved with Crank-Nicolson and Thomas algorithm. Full Greeks via finite-difference bumps.

```
Price: $16.60  |  Δ=0.0637  |  Θ=9.10  |  Vega=57.53  |  Rho=67.68
```

**0B · European Floating-Strike Monte Carlo**  
M=20,000 paths, antithetic variates, geometric Asian control variate. 2.23% gap vs CN.

```
Price: $16.24 ± $0.11  (SE = 0.69% of option value)
```

---

### Extensions

**1 · Running-Sum Reformulation**  
The original American ADI failed because `(S−A)/t → ∞` as `t → 0`. The fix: switch state variables to `I_t = ∫S_u du`, giving `dI/dt = S` — bounded for all t. Documents three remaining challenges that prevent a naive CN-ADI implementation even after the reformulation (upwind direction, Imax sizing, ADI+penalty interaction). American pricing uses LS-MC instead.

```
American (T=90d, K=ATM):  $8.64 ± $0.05
European lower bound:      $7.83
Early-exercise premium:    $0.81 (10.3%)
```

**2 · Convergence Study**  
Empirical second-order convergence confirmed (measured order: 2.37). Richardson-extrapolated reference: $16.6464.

**3 · Implied Volatility Surface**  
Round-trip IV inversion via Brent bisection. Zero inversion error across all 54 grid points (9 vol levels × 6 maturities), confirming the CN pricer is monotone in σ.

**4 · Longstaff-Schwartz — American Asian**  
Basis functions `[1, S/K, A/K, (S/K)², (A/K)², S·A/K²]` scaled for conditioning. Degrades to `[1, A/K, (A/K)²]` when S≈A to avoid singular regression. Uses `lstsq` with ridge `1e-4` rather than solving the normal equations directly.

```
T=1yr:  American $18.30 ± $0.11  vs  European $16.40 ± $0.10  (premium $1.90)
T=90d:  American $8.67 ± $0.05   vs  European $7.76 ± $0.05   (premium $0.91)
```

**5 · Fixed-Strike Asian Option**  
Payoff `max(A_T − K, 0)` priced over a 7×3 moneyness-maturity grid. Put-call parity error: $0.07 (1.7% — within MC noise).

**6 · Discrete vs Continuous Averaging Gap**  
The CN PDE assumes continuous averaging; MC uses discrete fixings. Gap follows O(1/N) broadly but is largest at short maturities: −$5.53 at T=0.25yr with 13 weekly fixings, versus −$0.63 at T=1yr with 52.

---

## Results Summary

| Section | Method | Key Output |
|---------|--------|-----------|
| 0A | CN 1D (Vécer) | $16.60, full Greeks |
| 0B | MC + control variate | $16.24 ± $0.11, 2.23% vs CN |
| 1 | Running-sum + LS-MC | Premium $0.81 on 90d American |
| 2 | Convergence | Order 2.37 — consistent with O(h²) |
| 3 | IV surface | 0.00000 round-trip error |
| 4 | Longstaff-Schwartz | Premium $1.90 on 1yr American |
| 5 | Fixed-strike grid | PCP error $0.07 |
| 6 | Discrete-continuous gap | −$5.53 at T=0.25, shrinks with N |

---

## Installation

```bash
git clone https://github.com/<your-username>/asian-option-pricing.git
cd asian-option-pricing
pip install -r requirements.txt
```

**requirements.txt**
```
numpy
pandas
scipy
matplotlib
yfinance
```

---

## Usage

Open `asian_options_extended.ipynb` in Jupyter or Colab. Run cells in order — market data is fetched live at startup and reused throughout.

For Colab:
```python
# Uncomment in cell 1:
# !pip install yfinance --quiet
```

---

## Known Limitations

**2D American PDE (Section 1):** The running-sum reformulation is theoretically correct but the full CN-ADI implementation requires:
- Forward upwind differencing in the I-dimension (information flows from large I to small I in backward time)
- Imax sized to `K·T` (not `S_max·T`)
- An asymptotic upper BC at `S_max` rather than Neumann extrapolation
- Robust handling of the American constraint with ADI splitting

The current implementation prices American Asians via Longstaff-Schwartz, which is the industry standard for this product.

---

## References

1. Vécer, J. (2001). A new PDE approach for pricing arithmetic average Asian options. *Journal of Computational Finance*, 4(4), 105–113.
2. Longstaff, F.A. and Schwartz, E.S. (2001). Valuing American options by simulation. *Review of Financial Studies*, 14(1), 113–147.
3. Crank, J. and Nicolson, P. (1947). A practical method for numerical evaluation of solutions of PDEs. *Proc. Cambridge Philosophical Society*, 43(1), 50–67.
4. Rannacher, R. (1984). Finite element solution of diffusion problems with irregular data. *Numerische Mathematik*, 43(2), 309–327.
5. Glasserman, P. (2003). *Monte Carlo Methods in Financial Engineering*. Springer-Verlag.
6. Broadie, M., Glasserman, P. and Kou, S.G. (1999). Connecting discrete and continuous path-dependent options. *Finance and Stochastics*, 3(1), 55–82.

---

## Author

**Karan Parihar** — MQF Candidate, Rutgers University  
[LinkedIn](https://linkedin.com/in/<your-handle>) · [GitHub](https://github.com/<your-username>)
