# Luna Stock Lab Back Test v1

## Goal

Turn the current discretionary "爆升盤" workflow into a rules-based framework that can be replayed on historical US stock data.

This version is designed for:

- post-earnings momentum
- catalyst-driven breakouts
- strict risk control

This is not a pure quant factor model yet. It is the first ruleset that approximates the current decision style closely enough for historical testing.

## Universe

- Market: US equities only
- Liquidity filter:
  - close price >= USD 10
  - 20-day average daily dollar volume >= USD 20,000,000
- Exclude:
  - ETFs
  - ADRs with very low liquidity
  - stocks with pending delisting risk

## Trade Types

### Setup A: Post-Earnings Momentum

Use when a company has reported earnings within the last 1 trading day.

Entry candidate only if all conditions are met:

1. Revenue beat or guidance raise or both
2. Day-0 price reaction > +4%
3. Day-0 volume >= 1.8x 20-day average volume
4. Day-0 close is in top 25% of intraday range
5. Gap-up from previous close is not greater than 12%
6. Market regime is not risk-off

### Setup B: Catalyst Breakout

Use when there is a hard catalyst:

- analyst target hike
- major product / AI / regulatory / partnership announcement
- sector-wide momentum trigger

Entry candidate only if all conditions are met:

1. Daily gain between +3% and +9%
2. Volume >= 2.0x 20-day average volume
3. Close above 20-day high
4. Relative strength vs S&P 500 over last 5 trading days is positive
5. Stock is not more than 18% above 20-day moving average

## Market Regime Filter

Allow new long entries only if at least 2 of the following are true:

1. S&P 500 closes above 20-day moving average
2. Nasdaq 100 closes above 20-day moving average
3. More than 50% of stocks in watch universe close above 20-day moving average
4. VIX is below 22

If fewer than 2 are true:

- no new long positions
- existing positions can only be reduced, held, or stopped out

## Ranking Rules

If more than 2 names qualify on the same day, rank them using:

1. Guidance raise beats simple earnings beat
2. Higher volume expansion ranks higher
3. Stronger close-in-range ranks higher
4. Lower extension from 20-day moving average ranks higher
5. AI / semiconductor / software infra themes get tie-break priority in risk-on regime

Keep only top 2 names as the daily research list.

## Entry Rules

Daily system output can only be:

- `BUY`
- `SELL`
- `HOLD`
- `NO TRADE`

### Buy

Buy only if:

1. Name passed Setup A or Setup B
2. Name is in top 2 ranking for the day
3. Reward-to-risk >= 2.0 based on initial target and stop
4. Total active portfolio exposure after entry stays within limit

### No Trade

No trade if:

- no candidate meets rules
- risk-off regime blocks entries
- setup is too extended
- reward-to-risk is below 2.0

## Position Sizing

Portfolio base: HKD 100,000 simulated capital

### Default risk unit

- Max risk per trade: 1.0% of equity
- Max concurrent exposure: 35% of equity
- Max single position size:
  - Setup A: 25% of equity
  - Setup B: 15% of equity

Position size formula:

`shares = floor((equity * 0.01) / (entry_price - stop_price))`

Then cap by max position size.

## Exit Rules

### Hard Stop

Exit immediately if price closes below stop.

Initial stop:

- Setup A: lower of
  - earnings reaction day low
  - 6% below entry
- Setup B: lower of
  - breakout day low
  - 7% below entry

### Profit Taking

Scale logic:

1. Sell 50% at +2R
2. Move stop on remaining size to breakeven
3. Exit remaining position if:
   - price closes below 5-day moving average, or
   - price closes below prior day low after a 3-day run, or
   - target extension reaches +4R

### Time Stop

Exit full position after 5 trading days if:

- price has not reached +1R
- or momentum has clearly stalled

## Portfolio Rules

- Maximum 2 open positions at one time
- Do not open 2 names from the exact same low-liquidity sub-theme
- If 2 qualified names are highly correlated, prefer the stronger one only

## Back Test Output Metrics

Run the back test on 1-year, 2-year, and 3-year windows.

Track:

- total return
- CAGR if period > 1 year
- win rate
- average winner
- average loser
- profit factor
- average hold days
- max drawdown
- exposure rate
- number of trades
- return by setup type
- return by market regime

## Required Historical Data

Minimum:

- daily OHLCV
- earnings dates
- earnings surprise / guidance flag
- analyst target-change events if available
- benchmark index data: S&P 500, Nasdaq 100, VIX

Nice to have:

- intraday range quality
- news event tags
- sector classification

## Known Limitations

This v1 still simplifies several discretionary judgments:

- "消息夠唔夠硬" is approximated by hard event tags
- theme quality is still coarsely scored
- narrative durability is not fully modeled
- no slippage / spread model yet

So v1 should be treated as:

- a truth-check against story bias
- a filter for bad habits
- a first pass before v2 factor refinement

## Next Step

If v1 back test results are acceptable, build v2 with:

- slippage assumptions
- sector-relative strength scoring
- implied volatility / earnings-move normalization
- market breadth factor
- regime-specific parameter tuning
