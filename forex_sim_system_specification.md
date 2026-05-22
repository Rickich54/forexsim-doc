# Forex News Trading Simulation — System Specification
**Document Type:** Technical Product Specification  
**Version:** 1.0  
**Classification:** Internal — Product & Engineering  

---

## DELIVERABLE 1 — DATA REQUIREMENTS SPECIFICATION

---

### 1.1 Economic Event Data

| Field | Description | Format | Update Frequency | Purpose in Simulation |
|---|---|---|---|---|
| `event_id` | Unique identifier per economic release | UUID / string | Per event | Primary key for all event-linked records |
| `event_name` | Canonical name of release (e.g. "US Non-Farm Payrolls") | string | Per event | Display label; links to historical data |
| `currency_pair` | Primary FX pair affected (e.g. `EURUSD`, `USDJPY`) | string [6] | Per event | Routes event to correct simulation instrument |
| `release_datetime` | Scheduled UTC timestamp of release | ISO 8601 timestamp | Per event | Triggers countdown timer in simulation |
| `forecast_value` | Analyst consensus estimate prior to release | float | Per event (pre-release) | Anchors player expectations; used in surprise calc |
| `actual_value` | Published figure at release time | float | At release | Drives price impact engine |
| `previous_value` | Prior period's released figure | float | Per event | Context for revision analysis |
| `revision_flag` | Boolean: prior period figure was revised | boolean | Per event | Adjusts impact weighting in engine |
| `surprise_magnitude` | `(actual - forecast) / std_dev` z-score | float | At release | Core input to price impact model |
| `event_tier` | Importance classification: Tier 1 / 2 / 3 | enum | Per event | Scales simulation volatility and scoring weight |
| `country_code` | ISO 3166-1 alpha-2 issuing country | string [2] | Per event | Filters events by player-selected region |
| `data_category` | Classification: Inflation / Employment / Growth / Trade | enum | Per event | Used in educational tagging and skill tracking |

---

### 1.2 FX Price Data

| Field | Description | Format | Update Frequency | Purpose in Simulation |
|---|---|---|---|---|
| `instrument` | FX pair symbol (e.g. `GBPUSD`) | string [6] | Real-time | Identifies the traded instrument |
| `bid_price` | Best buy price from market maker | float (5 decimals) | Real-time (tick) | Spread calculation; player entry price on sell |
| `ask_price` | Best sell price from market maker | float (5 decimals) | Real-time (tick) | Spread calculation; player entry price on buy |
| `mid_price` | `(bid + ask) / 2` | float (5 decimals) | Real-time (tick) | Chart display; P&L reference price |
| `spread_pips` | `ask - bid` in pips | float | Real-time | Slippage modeling; cost of trade |
| `ohlc_1m` | Open, High, Low, Close at 1-minute resolution | float[4] | Per minute | Candlestick chart rendering |
| `ohlc_5m` | OHLC at 5-minute resolution | float[4] | Per 5 min | Wider context chart |
| `volume_proxy` | Tick count or lot volume estimate | integer / float | Per minute | Liquidity indicator; spread widening trigger |
| `timestamp_utc` | UTC timestamp of price record | ISO 8601 timestamp | Per tick | Time-axis alignment with event releases |
| `data_source_id` | Originating feed identifier | string | Per tick | Audit trail; fallback routing |

---

### 1.3 Market Context Data

| Field | Description | Format | Update Frequency | Purpose in Simulation |
|---|---|---|---|---|
| `atr_14` | 14-period Average True Range | float | Per bar (1m/5m) | Baseline volatility; scales TP/SL suggestions |
| `implied_volatility` | Options-derived 1-month IV for instrument | float (%) | Daily | Pre-event risk overlay in simulation |
| `vix_index` | CBOE VIX as global risk proxy | float | Real-time | Risk-off / risk-on regime flag |
| `correlation_matrix` | Pair-wise 30-day rolling correlation between FX pairs | float[N×N] | Daily | Multi-instrument signal validation |
| `cot_net_position` | CFTC Commitment of Traders net speculative position | integer (contracts) | Weekly | Positioning context for educational layer |
| `risk_regime` | Computed flag: Risk-On / Risk-Off / Neutral | enum | Per event | Adjusts simulation price response model |
| `event_window_vol` | Realised vol in ±15 min around prior events | float | Per event | Calibrates countdown volatility warnings |
| `liquidity_session` | Active trading session: London / NY / Tokyo / Overlap | enum | Hourly | Session filter for replay mode selection |
| `economic_calendar_surprise_history` | Rolling z-score of past N surprises per event type | float | Per event | Trains player on average reaction benchmarks |

---

### 1.4 Player and Game Data

| Field | Description | Format | Update Frequency | Purpose in Simulation |
|---|---|---|---|---|
| `player_id` | Unique player identifier | UUID | On registration | Primary key for all player records |
| `session_id` | Unique identifier per game session | UUID | Per session | Groups trades and events within one game round |
| `virtual_balance` | Player's current simulated account balance | float | Per trade | P&L tracking; game-over trigger |
| `trade_id` | Unique identifier per submitted order | UUID | Per trade | Links entry, exit, P&L, and score records |
| `trade_direction` | Long or Short | enum | Per trade | Determines P&L sign |
| `entry_price` | Price at which the trade was opened | float | At trade open | P&L base |
| `exit_price` | Price at which the trade was closed | float | At trade close | P&L realisation |
| `position_size_lots` | Notional trade size in standard lots | float | Per trade | Scales P&L and margin requirement |
| `stop_loss_price` | Player-defined SL level | float | Per trade | Risk manager input; auto-close trigger |
| `take_profit_price` | Player-defined TP level | float | Per trade | Auto-close trigger |
| `trade_pnl_pips` | Realised P&L in pips | float | At trade close | Core scoring input |
| `trade_pnl_usd` | Realised P&L in base currency | float | At trade close | Balance update |
| `reaction_time_ms` | Time between event release and player order submission | integer (ms) | Per trade | Evaluated in educational debrief |
| `player_score` | Cumulative simulation score (multi-dimensional) | float | Per session | Leaderboard ranking |
| `accuracy_rating` | % of trades directionally aligned with event outcome | float (%) | Per session | Skill tracking metric |
| `events_played` | Count of economic events traded in career | integer | Cumulative | Progression gating |
| `preferred_pairs` | Player's most-traded FX pairs | string[] | Per session | Personalisation engine |
| `difficulty_level` | Current simulation difficulty tier | enum | Per session | Scales forecast error, spread, and time pressure |

---

### 1.5 System and Operational Data

| Field | Description | Format | Update Frequency | Purpose in Simulation |
|---|---|---|---|---|
| `event_trigger_log` | Record of every game event fired with timestamp | JSON / log | Per event | Audit and replay reconstruction |
| `api_latency_ms` | Round-trip latency for each external data call | integer (ms) | Per API call | Health monitoring; switches to historical cache |
| `data_staleness_flag` | Boolean: data older than threshold | boolean | Real-time | Degrades to replay mode gracefully |
| `simulation_tick` | Internal simulation clock (not wall time) | integer | Per tick | Replay speed control |
| `scoring_version` | Algorithm version used in session scoring | string | Per session | Reproducibility for competitive modes |
| `error_code` | Categorised system error if data fetch fails | enum / string | On error | Triggers fallback data routing |
| `leaderboard_snapshot` | Top-N player rankings at session end | JSON | Per session end | Rendered in presentation layer |

---

## DELIVERABLE 2 — DATA FLOW ARCHITECTURE

*See the visual diagram above for the six-layer layout. The step-by-step flow is described below.*

---

### Step-by-Step Data Flow

**Step 1 — External data ingestion**

Economic calendar APIs (Investing.com, Trading Economics, FRED) push scheduled event metadata into the ingestion layer on a polling cycle of every 15 minutes or via webhook on event release. FX price feeds (Twelve Data, Alpha Vantage, or a broker WebSocket) stream tick and OHLC data in real-time. Volatility, sentiment, and central bank data are fetched on daily or per-event schedules.

**Step 2 — Processing layer transforms raw data**

- The **Event Processor** computes `surprise_magnitude = (actual - forecast) / historical_std_dev` the instant an actual figure is received.
- The **Price Impact Engine** applies a parametric model: `expected_move_pips = surprise_magnitude × sensitivity_coefficient × liquidity_session_multiplier`.
- The **Volatility Calculator** derives ATR and implied volatility regime flags.
- The **Sentiment Scorer** runs NLP classification on headline text (bullish / bearish / neutral) and blends with quantitative surprise.
- All output is normalised to a common schema and timestamped in UTC before storage.

**Step 3 — Data written to storage**

- Time-series data (prices, vol) writes to a time-series database (InfluxDB or TimescaleDB equivalent).
- Event metadata, surprise history, and outcomes write to a relational store (PostgreSQL equivalent).
- Player state and session data write to a fast key-value store (Redis equivalent) for low-latency access during gameplay.
- Historical event archives support the replay engine with pre-computed outcomes.

**Step 4 — Simulation engine consumes stored data**

- On event countdown start, the engine reads the scheduled event record and begins a visual countdown in the UI.
- At T=0 (release), the engine reads the processed surprise value, queries the price impact model, and applies a simulated price move to the live chart.
- Player orders submitted during the pre-announcement window or on the spike are received by the Trade Order Handler.
- The P&L Calculator settles each trade using bid/ask spreads plus simulated slippage.
- The Risk Manager enforces SL/TP logic and margin calls.
- In historical replay mode, the Replay Engine substitutes stored historical prices and outcomes for real-time feeds.

**Step 5 — Results surface in the presentation layer**

- The live price chart updates from the simulation engine's internal tick feed.
- The news event panel displays the event name, forecast, actual, and surprise magnitude at release.
- The trade blotter shows open and closed positions with real-time P&L.
- At session end, the leaderboard updates from the scoring store.

**Step 6 — Educational feedback layer processes session analytics**

- The post-trade debrief compares the player's trade direction and timing against the optimal response derived from the price impact model.
- Reaction time, directional accuracy, and risk-reward ratio are scored and logged.
- Bias detection flags systematic patterns (e.g. always trading against the trend, always exiting too early).
- Skill progression unlocks new instruments, higher-tier events, or tighter spreads.
- All session data is written back to the player store, closing the loop.

---

### Architecture Layer Summary

```
┌─────────────────────────────────────────────────────────────────────┐
│  LAYER 1 — DATA INGESTION                                           │
│  Economic calendar │ FX price feed │ Vol/VIX │ News │ Central bank  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ raw data
┌──────────────────────────────▼──────────────────────────────────────┐
│  LAYER 2 — PROCESSING                                               │
│  Event processor │ Price impact engine │ Vol calc │ NLP scorer      │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ normalised events
┌──────────────────────────────▼──────────────────────────────────────┐
│  LAYER 3 — STORAGE                                                  │
│  Time-series DB │ Historical replay │ Event metadata │ Player store  │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ event triggers + player state
┌──────────────────────────────▼──────────────────────────────────────┐
│  LAYER 4 — SIMULATION ENGINE                                        │
│  Event trigger │ Order handler │ P&L calc │ Risk manager │ Replay   │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ player results + metrics
┌──────────────────────────────▼──────────────────────────────────────┐
│  LAYER 5 — PRESENTATION                                             │
│  Live chart │ News panel │ Trade blotter │ P&L dash │ Leaderboard   │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ session analytics
┌──────────────────────────────▼──────────────────────────────────────┐
│  LAYER 6 — EDUCATIONAL FEEDBACK                                     │
│  Post-trade debrief │ Reaction analysis │ Bias detect │ Progression  │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ feedback loop (writes back to player store)
                           └──────────────────────────────────► LAYER 4
```

---

## DELIVERABLE 3 — DATA SOURCE MAPPING

---

### A. Economic Calendar Data Sources

| Platform | URL | Data Provided | Access Method | Free Tier | Simulation Role |
|---|---|---|---|---|---|
| Investing.com | https://www.investing.com/economic-calendar | Global economic events with forecast, actual, previous, and tier rating | Web scraping (unofficial), paid API via RapidAPI | Scraping: free; API: paid | Primary economic calendar. Provides event schedule, consensus forecasts, and actuals at release — core input to the surprise engine. |
| Trading Economics | https://tradingeconomics.com/calendar | 300+ country economic calendars, forecasts, historical actuals | REST API + CSV | Free tier: 2 API calls/sec, limited history; paid from $49/mo | Secondary calendar source. Strong on emerging market data. Use for coverage of RBI, BCB, and non-G10 events. |
| Myfxbook | https://www.myfxbook.com/forex-economic-calendar | Forex-specific economic calendar | Web scraping | Free | Supplementary validation source. Useful for community-sourced tier ratings and expected impact labels. |
| ForexFactory | https://www.forexfactory.com/calendar | Retail-focused FX economic calendar with impact ratings | Web scraping | Free | Historical replay population. Widely used by retail traders — mirrors player expectations well. |

---

### B. FX Price Data APIs

| Platform | URL | Data Provided | Access Method | Free Tier | Simulation Role |
|---|---|---|---|---|---|
| Alpha Vantage | https://www.alphavantage.co | Forex OHLCV at 1m, 5m, 15m, daily, weekly; real-time quotes | REST API (JSON) | Free: 25 req/day; Premium from $50/mo | Historical OHLCV for replay engine. Download pre-event and post-event windows for all major pairs. |
| Twelve Data | https://twelvedata.com | Real-time and historical forex tick, OHLCV, indicators | REST API + WebSocket | Free: 800 API credits/day; paid from $29/mo | Real-time price feed for live simulation mode. WebSocket stream delivers 1-second updates per instrument. |
| OANDA API | https://developer.oanda.com | Institutional-grade bid/ask streaming, OHLC, order book | REST + Streaming API | Free with demo account | Bid/ask spread data. Best source for realistic spread simulation across session types. Requires account creation. |
| Polygon.io | https://polygon.io/forex | Forex tick, OHLCV, real-time quotes, WebSocket | REST + WebSocket | Free: 5 API calls/min; paid from $29/mo | Fallback real-time data source. Supports multi-instrument streaming for portfolio simulation modes. |
| ExchangeRate-API | https://www.exchangeratesapi.io | Daily mid-market FX rates, 170 currencies | REST API | Free: 250 req/month | Auxiliary rate conversion. Used for normalising P&L across base currencies. |

---

### C. Volatility and Macro Data Sources

| Platform | URL | Data Provided | Access Method | Free Tier | Simulation Role |
|---|---|---|---|---|---|
| FRED (Federal Reserve) | https://fred.stlouisfed.org | Thousands of US and global macro series: CPI, PCE, unemployment, GDP, yield curves, MOVE index | REST API (JSON/XML) | Fully free — API key required (free) | Core macro context layer. Feeds historical economic condition overlays into replay mode. FRED series `VIXCLS` provides historical VIX. |
| CBOE | https://www.cboe.com/tradable_products/vix | VIX real-time and historical data | Website download (CSV) | Free (manual) | Daily VIX import for risk regime classification in simulation. Signals risk-off environments where spreads widen. |
| World Bank Open Data | https://data.worldbank.org | GDP growth, inflation, current account, trade balance for 200+ countries | REST API | Fully free | Macro context for emerging market events. Used in educational layer to explain structural economic conditions. |
| IMF Data | https://data.imf.org | Balance of payments, exchange rates, direction of trade statistics | REST API + manual download | Fully free | Long-run structural data. Supports educational content on currency fundamentals. |
| BIS (Bank for International Settlements) | https://www.bis.org/statistics | FX market turnover, dealer positions, cross-border banking statistics | Manual download (CSV) | Fully free | Liquidity context data. Used to calibrate session-based spread multipliers. |

---

### D. Sentiment and News Data Sources

| Platform | URL | Data Provided | Access Method | Free Tier | Simulation Role |
|---|---|---|---|---|---|
| NewsAPI | https://newsapi.org | Real-time and historical news headlines from 150,000+ sources | REST API | Free: 100 req/day, 1-month history; paid from $449/mo | News headline stream for NLP sentiment scorer. Fetches headlines in the ±30 min window around each event. |
| GDELT Project | https://www.gdeltproject.org | Global event database with tone and sentiment scores, updated every 15 minutes | REST API + BigQuery | Fully free | Macro sentiment context layer. GDELT Tone scores can proxy broader market mood independent of individual releases. |
| MediaStack | https://mediastack.com | Real-time global news API with language and category filters | REST API | Free: 500 req/month | Backup sentiment source. Used when NewsAPI quota is exhausted. |
| Reddit (via Pushshift / official API) | https://www.reddit.com/dev/api | Forum sentiment from r/Forex, r/economics; useful as retail positioning proxy | REST API | Free (rate-limited) | Retail positioning and fear/greed sentiment proxy. Educational layer uses subreddit tone to illustrate herd behaviour. |

---

### E. Central Bank and Official Data Sources

| Platform | URL | Data Provided | Access Method | Free Tier | Simulation Role |
|---|---|---|---|---|---|
| US Federal Reserve | https://www.federalreserve.gov/data.htm | FOMC meeting dates, statements, minutes, dot plot, Beige Book | Manual + RSS | Fully free | Primary source for Fed rate decisions — highest-impact events in simulation. Meeting calendars populate the event schedule. |
| European Central Bank (ECB) | https://www.ecb.europa.eu/stats | ECB policy decisions, HICP, M3, TARGET2, exchange rates | REST API + CSV | Fully free | EUR event source. ECB statistical data warehouse API provides granular euro area economic releases. |
| Bank of England (BoE) | https://www.bankofengland.co.uk/statistics | MPC decisions, CPI, GDP, trade, credit conditions | REST API + manual | Fully free | GBP event source. BoE statistics API delivers historical policy rate and inflation series for replay calibration. |
| Reserve Bank of India (RBI) | https://www.rbi.org.in/scripts/PublicationsView.aspx | Policy rates, inflation, trade, forex reserves | Manual download | Fully free | INR simulation pair support. Useful for emerging market difficulty tiers. |
| Bank of Japan (BoJ) | https://www.boj.or.jp/en/statistics | Policy rate decisions, Tankan survey, CPI, trade balance | Manual download + RSS | Fully free | JPY event source. BoJ decisions drive high-volatility JPY events; replay archive populated from press release timestamps. |
| Statistics Canada | https://www150.statcan.gc.ca/n1/en | CPI, employment, GDP, trade — all Canadian macro releases | REST API | Fully free | CAD event source. StatCan API delivers scheduled release dates and actuals for all CAD-impacting data. |

---

*End of specification. This document is intended for ingestion by the product and engineering teams as a starting point for sprint planning, API procurement, and database schema design.*
