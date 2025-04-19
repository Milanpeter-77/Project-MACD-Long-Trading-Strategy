# **MACD Trading Strategy for Long Positions**  

This analysis implements a MACD-based trading strategy focused exclusively on long positions. It evaluates historical performance using backtests that ignore initial capital, focusing purely on returns. By applying grid search across a range of MACD parameter combinations, the strategy identifies the most effective settings based on historical data – offering a systematic, data-driven approach to optimization.

---

## **MACD Strategy with Performance Metrics Function**

The core objective of my code is to implementa a **rule-based MACD trading strategy** on historical stock price data and compute a comprehensive set of performance metrics. It simulates trading behavior using realistic assumptions (such as slippage via open-price entries and transaction costs), enabling robust and reproducible backtesting.

### **Indicator Construction**

The strategy begins with the construction of the **MACD indicator**, a trend-following momentum oscillator:

- A **fast EMA** (default: 12) and a **slow EMA** (default: 26) are calculated from the **Open** prices.
    - *I use the **Open** price for strategy execution to simulate same-bar trading.*
    - *Since MACD is based on Open prices, signals can be computed pre-market, enabling immediate execution at the bar's open.*
- The **MACD line** is the difference between these two EMAs.
- A **Signal line** (default: 9) is applied to the MACD to smooth it.
- The **MACD Histogram**, used for trading signals, is the difference between the MACD line and its Signal line.

After used for calculations, all indicators are rounded for clearer visualization and consistency.

### **Signal Logic & Position Tracking**

Trading signals are derived from the histogram:

- A **positive histogram** implies bullish momentum → **enter long**.
- A **negative histogram** implies bearish momentum → **exit**.

Trades are detected via changes in histogram polarity. A position is held if the previous histogram was positive, adding robustness to signal continuity.

- **Position**: 1 if in a long position, 0 otherwise.
- **Trades**: +1 for entry, -1 for exit, 0 otherwise.

### **Execution Pricing Model**

To simulate realistic trade behavior, strategy trades are executed at the **Open** price of the bar where the signal appears:

- On **entry/exit**, the return is based on the **Open** price.
- When holding, return is based on the **Close**.
- Non-position days have no return contribution.

The model is flexible and can easily be adapted to worst/best-case scenarios using the **High** or **Low** prices.

### **Returns & Cost Adjustments**

- **Strategy return** is calculated only when in position, adjusted for transaction costs at every entry or exit.
- **Daily return** and **cumulative return** are computed for both the raw stock and the strategy.

All returns are expressed in percentage terms for intuitive comparison.

### **Performance Metrics**

The function outputs a rich set of metrics for evaluation and optimization:

- **Total return** and **average return** of the strategy
- **Volatility** of returns (standard deviation)
- **Number of trades** executed
- **Return efficiency** (return per trade)
- **Win rate**: proportion of trades that are profitable
- **Max drawdown**: deepest equity loss from a peak
- **Sharpe ratio**: annualized risk-adjusted return

Each trade’s return is isolated based on entry/exit index pairs, allowing for clear aggregation of performance per position.

### **Return Formatting for Plotting**

For easy visualization:

- All percentage-based values (returns, drawdown, etc.) are multiplied by 100 and rounded.
- Price-based columns (Open, Close, EMAs, etc.) are rounded to two decimal places.

### **Why This Matters**

This function goes beyond simple signal generation — it offers a **realistic, interpretable, and extensible framework** for backtesting:

- Built-in **transaction cost modeling**
- **Precise trade tracking and return simulation**
- Rich **metric set for optimization and benchmarking**
- Clean, extensible logic suitable for layering filters, experimenting with slippage, or integrating into ML workflows

It is especially useful for those seeking **capital-agnostic, systematic evaluation** of momentum-based strategies.

## **Chart Overview: Price Action & MACD Indicator**

The first chart is a two-panel visualization combining **price movement** and **momentum analysis**:

1. **Top Panel – OHLC & Moving Averages**  
   Displays the asset’s candlestick data (Open, High, Low, Close), overlaid with:
   - A **Fast EMA** (short-term trend)
   - A **Slow EMA** (long-term trend)  
   This helps visualize price trends and potential crossovers that drive the MACD.

2. **Bottom Panel – MACD Indicator**  
   Shows the:
   - **MACD Line** (difference between fast and slow EMA)
   - **Signal Line** (EMA of MACD)
   - **MACD Histogram** (MACD minus Signal)  
   It captures momentum shifts and potential buy/sell signals.

A **zero line** in the MACD subplot highlights when the MACD changes sign — key for identifying bullish or bearish momentum.

<img width="997" alt="ohclmacd_crwd" src="https://github.com/user-attachments/assets/c608def3-93ca-40df-a521-130b825c5da1" />

## **Chart Overview: Strategy vs. Market Performance**

This chart visualizes the **cumulative returns** of the MACD trading strategy compared to a simple **buy-and-hold equity approach**:

- **Strategy Performance** and **Equity Performance** lines show how each evolves over time.
- **Open Position markers** indicate when the strategy enters a trade.
- **Close Position markers** show when the trade is exited.

A **horizontal zero line** helps assess when the strategy or equity is in profit or loss territory.  
An accompanying annotation summarizes key performance metrics (e.g., total return, win rate, Sharpe ratio), giving a quick snapshot of the strategy’s effectiveness.

<img width="997" alt="strategy_crwd" src="https://github.com/user-attachments/assets/709ae51f-ca30-4d78-8969-21b2475af17e" />

## **MACD Strategy Optimization: Parameter Grid Heatmap Analysis**

This chart presents a comprehensive visual analysis of the MACD strategy's performance across a range of parameter configurations, allowing us to explore how the choice of **Fast EMA**, **Slow EMA**, and **Signal EMA** values impacts key performance metrics.

**Structure**
The chart is organized as a **3x3 matrix of heatmaps**, where each subplot highlights a different performance or risk metric derived from the backtest results. These subplots share a common y-axis (**Fast EMA values**) and x-axis (**Slow EMA values**), forming a full grid of parameter combinations.

The metrics visualized are:

- **Top Row:**
  - **Total Return** – The cumulative return of the strategy over the full backtest period.
  - **Average Return** – The mean return per position.
  - **Volatility** – The standard deviation of returns per position, capturing the strategy’s variability or risk.

- **Middle Row:**
  - **Total Return / Trade Count** – A measure of return efficiency, showing how much total return is generated per trade.
  - **Average Return / Trade Count** – Similar to the above, but normalized by average return instead of final outcome.
  - **Trade Count** – Total number of trades executed for each parameter set.

- **Bottom Row:**
  - **Maximum Drawdown** – The largest peak-to-trough loss observed, critical for evaluating downside risk.
  - **Sharpe Ratio** – A standard risk-adjusted return metric, showing return per unit of volatility.
  - **Win Rate** – The percentage of trades that were profitable.

**Interactive Features**
A **slider** along the top allows dynamic adjustment of the **Signal EMA** parameter. When the slider is moved, each heatmap is updated in real-time to reflect results for that specific signal value. This enables quick comparison across slices of the 3D parameter space (Fast × Slow × Signal) without overwhelming the user with separate plots.

**Color Mapping**
A custom colorscale is used across all heatmaps, with a visually intuitive progression from deep purple (low values) to bright yellow(high values), and finally red for the **top 1%** of values. This design highlights optimal zones immediately, making it easy to spot high-performing configurations. For metrics where *lower* is better (e.g. Volatility, Drawdown), the scale is reversed accordingly.

**Interpretation Aid**
To the right of the chart, a detailed annotation explains the meaning of each metric. These annotations break down complex financial terms into accessible definitions, guiding users in interpreting the charts whether they are analysts, developers, or stakeholders with limited financial background.

**Use Case**
This chart is ideal for **hyperparameter tuning**, offering a clear visual method for selecting parameter sets that balance **return, consistency, trade frequency, and risk**. By comparing all metrics simultaneously, it avoids the pitfalls of optimizing for a single metric at the expense of others.

<img width="999" alt="heatmap_crwd" src="https://github.com/user-attachments/assets/89ea15f0-eb5a-40c3-a2cc-4b6bf556dadc" />

**Animated version in a video:**

https://github.com/user-attachments/assets/f138bd70-f2d0-4fcc-bb3a-9482ec671cb4



