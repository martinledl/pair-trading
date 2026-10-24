# Pair Trading — Course Project Plan (v2)

**Professor's constraints:** one narrative-driven Jupyter notebook, no massive multi-file project. Answer three questions: *What problem did you solve? What did you use and why? How did it turn out?* Most projects are elementary — ours should be slightly above that without bloat.

**Goal:** Pairs trading that adapts to changing relationships (Kalman filter) and cuts losses mathematically when the relationship breaks (AR(1)/OU half-life time-stop).

---

## Design decisions (fixes from v1)

1. **Three-way backtest, not two-way.** v1 compared only (Kalman + hold) vs (Kalman + time-stop), which tests the time-stop but never tests whether Kalman helps. The backtest compares:
   1. *Static:* rolling-OLS hedge ratio, hold until reversion.
   2. *Dynamic:* Kalman hedge ratio, hold until reversion.
   3. *Smart:* Kalman hedge ratio + time-stop at $2\times$ half-life.
2. **No pre-written conclusion.** v1 assumed the smart strategy avoids a drawdown — verify on real data first. Choose a period where a structural break demonstrably hurts static betas (COVID crash, 2022 tech drawdown). If results disagree, present the honest result with analysis; a rigorous negative result grades better than a suspiciously clean win.
3. **No look-ahead bias.** Rolling z-score (not full-sample σ). Half-life re-estimated every ~21 days on a trailing window. Signals computed from data up to $t$, executed at $t+1$. Remaining simplifications stated in a *Limitations* cell.
4. **Correct statistical test.** With an estimated hedge ratio, the proper test is cointegration (Engle–Granger, `statsmodels.coint`); plain ADF is used as a complement, with one sentence acknowledging the distinction.
5. **Kalman tuning is planned, not improvised.** Grid search over the process/observation noise ratio ($Q/R$); rolling-OLS beta serves as visual sanity check.
6. **Readable loop over clever vectorization.** A plain `for` loop over ~1250 days is clearer, easier to debug, and shows the trade lifecycle. Vectorize only the P&L aggregation.

---

## Notebook structure

**1. Úvod & Problém (The Problem)**
* Classical pairs trading uses static linear regression, which fails when the relationship drifts. Textbook models hold losing trades forever after a structural break.
* Goal: dynamically adapt the hedge ratio and mathematically cut losses when the relationship breaks.

**2. Co a proč jsem použil (What & Why)**
* **Data:** 5y daily, two highly correlated ETFs (primary: `XLK` vs `QQQ`; backup: `SPY` vs `IWM` or `KO` vs `PEP` if the spread fails stationarity), `yfinance` with `auto_adjust=True`, log prices.
* **Static baseline:** 60-day rolling OLS beta → spread → ADF/cointegration → AR(1) half-life. This is the textbook model everything is measured against.
* **Kalmanův filtr:** 2-state random-walk model, state $z_t = (\alpha_t, \beta_t)$, $z_t = z_{t-1} + w_t$, observation $y_t = \alpha_t + \beta_t x_t + v_t$. Implemented by hand (~30 lines), tuned over a small $Q/R$ grid, plotted against rolling OLS beta.
* **AR(1) / Ornstein-Uhlenbeck half-life:** from $ds = \theta(\mu - s)\,dt + \sigma\,dW$, discretize to $s_t = c + \varphi s_{t-1} + \varepsilon_t$, so $\varphi = e^{-\theta}$ and half-life $= \ln 2 / (-\ln \varphi)$. The half-life becomes the mathematical time-stop (exit if a trade lasts more than $2\times$ half-life).

**3. Jak to dopadlo (The Results)**
* **Backtest:** the three-way comparison above. Entry at $\pm 2\sigma$ (rolling window), exit at $z = 0$, force-close after the time-stop. Metrics: total return, CAGR, Sharpe, max drawdown, # trades, win rate.
* **Visuals:** (1) equity curves with drawdown, (2) spread with entry/exit markers, (3) Kalman vs rolling-OLS beta, (4) rolling half-life.
* **Robustness:** threshold sensitivity (1.5σ / 2σ / 2.5σ), sample split (pre/post COVID). One paragraph on whether conclusions survive changing a knob.
* **Závěr (Conclusion):** maps back to the three questions; *Limitations* cell states remaining simplifications (no transaction costs, daily closes, execution assumptions).

---

## Roadmap (do it yourself, understand everything)

**Phase 0 — Prerequisites (½–1 day, no code)**
- On paper: derive the OU → AR(1) discretization and the half-life formula. This is the intellectual core of the project.
- Learn the Kalman filter at the 1D level, then the 2-state version. Resource: Roger Labbe, *Kalman and Bayesian Filters in Python*, ch. 1–3 (free, interactive).
- Checkpoint: write the 5 Kalman equations (predict, predict-covariance, innovation, gain, update) from memory.

**Phase 1 — Data (½ day)**
- Download, plot both series, correlation, scatter in log space.
- Checkpoint: explain *why* log prices and *why* the hedge ratio is a slope in log space.

**Phase 2 — Textbook baseline (1 day)**
- Rolling OLS beta → spread → stationarity tests → AR(1) half-life.
- Checkpoint: explain every number on the plots without notes.

**Phase 3 — Kalman (1–2 days)**
- Hand-rolled filter, tune $Q/R$ over a small grid, compare beta to rolling OLS.
- Checkpoint: explain the innovation and the Kalman gain intuitively, and what changing $Q$ does.

**Phase 4 — Backtest (1–2 days)**
- Plain loop, three strategies, metric table.
- Guard look-ahead (rolling z-score, trailing half-life, $t{+}1$ execution).
- Checkpoint: hand-trace one trade through the loop and verify the P&L.

**Phase 5 — Robustness + write-up (½–1 day)**
- Sensitivity checks, narrative polish, math derivations in markdown cells.

**Total: ~5–8 focused days.**

---

## AI usage protocol (learn, don't delegate)

- **AI as tutor, never author.** Allowed: "explain X", "why does this fail", "review my code and tell me what's wrong". Banned: "write my backtest".
- **Paper before keyboard.** Pseudocode on paper first, translate yourself. No pseudocode → not ready to code.
- **Closed-book rule.** After each phase, re-derive the key formula/loop from memory. If you can't, redo the reading — that is the exam skill.
- **Debug ritual.** Predict the bug before inspecting; form one hypothesis; test it with a print. Never paste an error into AI as a reflex.
- **Stuck budget.** After 2h stuck, a *conceptual hint* is allowed — never finished code.
- **Learning log** (`log.md`, 5 min/day): what clicked, what confused you. Becomes write-up material and oral-defense prep.
