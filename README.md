# Intraday VWAP Sell Execution Simulator

A simulation tool to execute optimal liquidation schedules for large equity blocks (e.g., 10% of daily volume) across an intraday session using Volume-Weighted Average Price (VWAP) benchmarks, historical volume profiles, and non-linear market impact modeling[cite: 1].

---

## Key Features

- **Historical Volume Profile:** Computes expected intraday volume distributions using multi-day lookbacks across 5-minute intervals[cite: 1].
- **Dynamic VWAP Tracking:** Dispatches child orders scaled to real-time bar volume[cite: 1].
- **Market Impact Model:** Estimates execution slippage and price shortfall via quadratic participation penalties[cite: 1].
- **Execution Diagnostics:** Evaluates benchmark tracking, participation rates, and profile deviations[cite: 1].

---

## Core Formulation

### 1. VWAP Benchmark
For typical price $P_t = \frac{\text{High}_t + \text{Low}_t + \text{Close}_t}{3}$ and bar volume $V_t$[cite: 1]:

$$\text{VWAP}_{\text{market}} = \frac{\sum P_t V_t}{\sum V_t} = \frac{\sum P_t V_t}{V_{\text{daily}}}$$
[cite: 1]

### 2. Child Order Sizing
For target order fraction $\alpha$ of total daily volume $V_{\text{daily}}$ ($Q = \alpha \times V_{\text{daily}}$), the per-bar target order size is[cite: 1]:

$$q_t = Q \times \frac{V_t}{V_{\text{daily}}} = \alpha \times V_t$$
[cite: 1]

Under a hard maximum participation cap $\rho_{\max}$[cite: 1]:

$$q_t^* = \min(q_t, \, \rho_{\max} \times V_t), \quad \rho_t = \frac{q_t^*}{V_t}$$
[cite: 1]

*Note:* If $\alpha = \rho_{\max}$, any shortfall from unexecuted volume cannot be recovered in later intervals without breaching the cap[cite: 1].

### 3. Non-Linear Market Impact
Temporary sell-side price impact $I_t$ (in bps) scales quadratically with participation[cite: 1]:

$$I_t(\text{bps}) = I_{\max} \times \left(\frac{\rho_t}{\rho_{\max}}\right)^2$$
[cite: 1]

$$\tilde{P}_t = P_t \times \left(1 - \frac{I_t}{10{,}000}\right)$$
[cite: 1]

### 4. Slippage & Performance Metrics
- **Execution VWAP:** $\text{VWAP}_{\text{exec}} = \frac{\sum \tilde{P}_t q_t^*}{\sum q_t^*}$[cite: 1]
- **Sell Shortfall (bps):** $\left(\frac{\text{VWAP}_{\text{market}}}{\text{VWAP}_{\text{exec}}} - 1\right) \times 10{,}000$[cite: 1]
- **Dollar Shortfall:** $(\text{VWAP}_{\text{market}} - \text{VWAP}_{\text{exec}}) \times \sum q_t^*$[cite: 1]

---

## Configuration

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `TICKER` | `str` | `'AAPL'` | Target equity ticker[cite: 1] |
| `PERIOD` | `str` | `'30d'` | Intraday download range[cite: 1] |
| `INTERVAL` | `str` | `'5m'` | Bar duration[cite: 1] |
| `TRADE_DATE` | `str`/`None` | `None` | Execution date (`YYYY-MM-DD`); `None` uses latest session[cite: 1] |
| `LOOKBACK_SESSIONS` | `int` | `15` | Historical sessions used for volume curve forecasting[cite: 1] |
| `ORDER_FRACTION_OF_DAILY_VOLUME` | `float` | `0.10` | Parent order size relative to total daily volume ($\alpha$)[cite: 1] |
| `MAX_PARTICIPATION_RATE` | `float` | `0.10` | Maximum volume participation allowed per bar ($\rho_{\max}$)[cite: 1] |
| `IMPACT_BPS_AT_MAX_PARTICIPATION` | `float` | `6.0` | Basis point price penalty at maximum participation[cite: 1] |

---

## Quickstart

### Requirements
pip install numpy pandas matplotlib seaborn yfinance

### Run
Open and run the notebook:
jupyter notebook traditional_vwap.ipynb

---

## Outputs

1. **Trade Summary Table:** Total volume, filled shares, completion rate, market VWAP, execution VWAP, and shortfall[cite: 1].
2. **Execution Charts:**
   - Price vs. Market VWAP and estimated fill prices[cite: 1].
   - Bar-by-bar participation vs. market volume[cite: 1].
   - Cumulative order completion vs. cumulative market volume[cite: 1].
   - Historical forecast curve vs. realized execution[cite: 1].
