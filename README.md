# Crypto Market Health Monitor

**Cross-asset monitoring of BTC, ETH and SOL using hourly Kraken market data**

A Python-based market-monitoring case study that turns hourly cryptocurrency data into interpretable signals for research and operational review. Rather than forecasting prices or issuing trading recommendations, the project identifies periods of unusual activity across **BTC/USD, ETH/USD and SOL/USD**, then examines how those periods differ from typical market conditions.

## Business problem

Crypto markets trade continuously. Reviewing every hourly observation across several assets is time-consuming, and a large price move in one asset does not necessarily indicate broader market activity. This prototype explores a practical question:

**Can a transparent, rule-based monitoring workflow narrow a continuous stream of market data into a manageable set of cross-asset episodes worth investigating?**

## Data and scope

- **Source:** Kraken public OHLC API; fixed CSV snapshots used for reproducibility.
- **Assets:** BTC/USD, ETH/USD and SOL/USD.
- **Period:** Approximately 31 August–30 September 2026.
- **Frequency:** Hourly; **721 observations per asset**.
- **Input fields:** OHLC prices, VWAP, trading volume and trade count.

The analysis describes the selected **Kraken USD trading pairs**, not total activity across all cryptocurrency exchanges.

## Methodology

1. **Prepare hourly market data:** load the frozen CSV files, parse timestamps and create derived indicators.
2. **Measure market conditions:** calculate hourly percentage price changes, absolute price changes and the rolling standard deviation of price changes over 24 observations.
3. **Define asset-specific thresholds:** use each asset's sample **90th percentile** for absolute price change, trading volume and rolling volatility.
4. **Assign market-health status:** count indicators exceeding their thresholds: **NORMAL** = 0, **WATCH** = 1, **ALERT** = 2 or 3.
5. **Detect synchronized activity:** align BTC, ETH and SOL by hour and count how many assets are simultaneously in ALERT.
6. **Group triple-ALERT observations:** create a new episode when the gap since the previous triple-ALERT observation exceeds **three hours**.
7. **Investigate episodes:** compare hourly price movements and estimated USD trading activity (`VWAP × volume`), normalized by each asset's median hourly USD activity.

Thresholds and the three-hour grouping rule are analytical design choices for this prototype; they are not industry-standard risk definitions.

## Key findings

| Measure | Result |
| --- | --- |
| Hourly observations per asset | **721** |
| Hours with no assets in ALERT | **649 (90.0%)** |
| Hours with exactly one asset in ALERT | **37** |
| Hours with exactly two assets in ALERT | **17** |
| Hours with all three assets in ALERT | **18 (2.5%)** |
| Distinct triple-ALERT episodes | **11** |

- Across individual assets, roughly **77%** of observations were classified as NORMAL; around **5–7%** were ALERT.
- Triple-ALERT observations clustered into **11 episodes**. **Episode 4 (11 September)** contained four triple-ALERT observations within a seven-hour span; this does **not** mean all seven hours were continuously in ALERT.
- ALERT episodes included **both rising and falling prices**. An ALERT is a signal of unusual conditions, **not** a prediction of price direction.
- Estimated USD trading activity was elevated during all 11 cross-asset episodes relative to each asset's typical hourly level. During **Episode 10**, the reported normalized activity was approximately **10.79× for BTC, 13.45× for ETH and 9.01× for SOL**.
- The cross-asset filter reduced **721 hourly observations** to **18 triple-ALERT hours** for targeted investigation.

These are **descriptive findings** from this dataset; they do not establish why the episodes occurred or whether they correspond to external market events.

## Product and operations relevance

A similar monitoring workflow could help market-data teams:

- Prioritize analyst review of synchronized, unusual market activity.
- Surface timestamps for deeper research against news or external event data.
- Build interpretable alert dashboards that explain which indicators triggered each status.
- Compare market behaviour across assets without treating their raw volatility and volume scales as equivalent.

This is a **research and monitoring prototype**, not a production surveillance system, validated risk model, or trading strategy.

## Project files

```text
Crypto_Market_Health_Monitor/
├── Prudence_Crypto_Market_Health_Monitor.ipynb
├── btc_kraken_market_data.csv
├── eth_kraken_market_data.csv
├── sol_kraken_market_data.csv
└── README.md
```

The notebook also includes **Appendix A**, documenting a reusable Kraken OHLC API collection function. The main analysis runs on the saved CSV snapshots so its historical window remains fixed.

## How to run

**Google Colab**

1. Open `Prudence_Crypto_Market_Health_Monitor.ipynb` in Google Colab.
2. Upload the three CSV files if prompted, keeping their filenames unchanged.
3. Select **Runtime → Run all**.
4. Review the market-health tables, cross-asset alert charts, episode analysis and conclusions.

**Local Jupyter environment**

1. Put the notebook and all three CSV files in the same working directory.
2. Install the required Python packages:

   ```bash
   pip install pandas numpy matplotlib requests notebook
   ```

3. Start Jupyter from the project directory and run the notebook from top to bottom.

The notebook searches the current working directory and standard Colab locations for the three CSV files. The `requests` package is used only if you choose to call the optional API collection function in the appendix.

## Limitations and possible extensions

- Only ~30 days of hourly data and three assets from one exchange were examined.
- Percentile thresholds are sample-dependent; different windows may produce different alerts.
- The three-hour episode grouping rule has not been externally validated.
- Co-occurring signals do not establish causation or prove the presence of a specific market event.
- Future work could evaluate longer periods, additional exchanges, alternative thresholds, and event annotations.

## Tools

**Python**, **pandas**, **NumPy**, **Matplotlib**, **Google Colab/Jupyter**, and the **Kraken public OHLC API**.

---

*Independent portfolio case study. Not affiliated with or endorsed by Kraken or Kaiko. For research and demonstration only; not financial advice.*
