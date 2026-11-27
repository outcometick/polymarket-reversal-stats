# Polymarket 5-minute Up/Down: how often does a 90–99% favourite lose?

[中文](README.zh-CN.md) · Data: [outcometick](https://outcometick.com?utm_source=github&utm_medium=readme&utm_campaign=reversal-stats) · Sister repo: [Predict.fun reversal stats](https://github.com/outcometick/predict-fun-reversal-stats)

In Polymarket's 5-minute crypto Up/Down markets, once one side's price first reaches 90%, 93%, 95%, 98% or 99%, how often does that side still lose at the close? This repository answers that from tick-level order-book data and **updates itself every day**.

**Coverage:** 2026-09-02 → 2026-11-25 (UTC), 85 days, 171,163 settled 5-minute markets across BTC, BNB, DOGE, ETH, HYPE, SOL, XRP. Last updated 2026-11-25.

## Headline — all assets, mid price

| favourite first reached | rounds that got there | favourite lost | reversal rate | 95% interval | avg price at trigger | win rate − price |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 90% | 169,277 | 15,508 | 9.16% | 9.03% – 9.30% | 0.9031 | 0.53 pp |
| 93% | 168,451 | 10,771 | 6.39% | 6.28% – 6.51% | 0.9318 | 0.42 pp |
| 95% | 166,787 | 7,393 | 4.43% | 4.33% – 4.53% | 0.9515 | 0.42 pp |
| 98% | 146,926 | 2,272 | 1.55% | 1.49% – 1.61% | 0.9808 | 0.37 pp |
| 99% | 16,864 | 68 | 0.40% | 0.32% – 0.51% | 0.9940 | 0.20 pp |

![realised vs implied reversal rate](charts/calibration.svg)

*Implied* is 1 − the average price paid at the trigger. When the realised rate sits above it, the favourite at that price lost more often than its price said; "win rate − price" is the same gap seen from the buyer's side, before fees.

## By asset — reversal rate at each threshold (mid)

| asset | 90% | 93% | 95% | 98% | 99% | rounds at 95% |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| **all** | 9.16% | 6.39% | 4.43% | 1.55% | 0.40% | 166,787 |
| BTC | 8.30% | 5.83% | 4.07% | 1.41% | 0.45% | 24,286 |
| BNB | 9.60% | 6.70% | 4.72% | 1.55% | 0.48% | 24,117 |
| DOGE | 9.43% | 6.61% | 4.52% | 1.60% | 0.40% | 24,116 |
| ETH | 8.68% | 5.88% | 3.98% | 1.53% | 0.68% | 24,277 |
| HYPE | 9.23% | 6.50% | 4.66% | 1.56% | 0.17% | 21,470 |
| SOL | 9.42% | 6.42% | 4.46% | 1.60% | 0.49% | 24,263 |
| XRP | 9.47% | 6.83% | 4.65% | 1.60% | 0.25% | 24,258 |

## By price basis — all assets

| threshold | mid | best bid | trade |
| --- | ---: | ---: | ---: |
| 90% | 9.16% | 8.24% | 7.49% |
| 93% | 6.39% | 5.54% | 5.35% |
| 95% | 4.43% | 3.67% | 4.04% |
| 98% | 1.55% | 1.15% | 2.05% |
| 99% | 0.40% | 0.57% | 1.30% |

## Over time

![weekly reversal rate](charts/weekly.svg)

## All the tables

- [`results/summary.csv`](results/summary.csv) — per asset × threshold: triggers, reversals, rate, 95% Wilson interval, average trigger price, win rate minus price
- [`results/by_price_basis.csv`](results/by_price_basis.csv) — the same, for each price basis
- [`results/daily.csv`](results/daily.csv) — per UTC day
- [`results/weekly.csv`](results/weekly.csv) — per week (Monday start, UTC)
- [`results/monthly.csv`](results/monthly.csv) — per month
- [`results/by_hour_utc.csv`](results/by_hour_utc.csv) — by the hour the market opened (UTC)
- [`results/by_seconds_to_close.csv`](results/by_seconds_to_close.csv) — by how many seconds before the close the threshold was first reached
- [`results/by_loser_peak.csv`](results/by_loser_peak.csv) — how high the losing side's mid price got before it lost
- [`results/data_quality.csv`](results/data_quality.csv) — per day and asset: rounds, settled, flat, unknown, rounds with no two-sided quote

Every table is computed from [`rounds/`](rounds/): one gzip CSV per UTC day, one row per market, with the outcome and — for each basis and threshold — which side got there first, when, and at what price. Use it to re-check a number or try your own thresholds.

## Definitions

- **Round**: one 5-minute market, assigned to the UTC day it opens.
- **Trigger**: the first moment inside the market's window when either side's price on the chosen basis is ≥ the threshold. That side is the favourite.
- **Reversal**: the favourite lost at settlement.
- Three price bases: **mid** = (best bid + best ask) / 2, **best bid**, and **trade** = the price of a print. Up and Down each have their own order book. Mid is counted only while both sides are quoted: a book with no ask is skipped rather than priced as 1.0. So a 99% mid needs an ask above 0.99, which only sub-cent ticks allow, and the 99% mid row has few rounds; for the 99% line read the best-bid basis.
- Only settled rounds count. Flat closes (rare) are left out.
- **Wilson interval**: 95% score interval on the reversal rate.
- **Win rate − price**: favourite win rate minus the average trigger price, in percentage points. Mid and best bid are not prices you can buy at (you pay the ask), and no fees are included — read it as a description of the market, not as a strategy result.

## Reproduce it

Code is in [`reversal_stats/`](reversal_stats/) (Python 3.10+, standard library only).

```bash
# one free sample day, no key
curl -L https://github.com/outcometick/polymarket-tick-data-samples/releases/latest/download/polymarket-data-samples.tar.gz | tar xz
python -m reversal_stats extract --venue polymarket --data ./polymarket-data-samples --day <sample day> --out my-rounds
python -m reversal_stats report --venue polymarket --rounds my-rounds --out my-report

# any range you have a data subscription for
pip install outcometick
export OT_KEY="ck_..."
python -m reversal_stats fetch --venue polymarket --from <first day> --to <last day> --data ./archive
python -m reversal_stats extract --venue polymarket --data ./archive --from <first day> --to <last day> --out my-rounds
```

Inputs per day: best_bid_ask (unthrottled top of book), last_trade_price and markets, as delivered by the [outcometick](https://outcometick.com?utm_source=github&utm_medium=readme&utm_campaign=reversal-stats) archive. The sample README names its day.

## Data and license

Market data: [outcometick](https://outcometick.com?utm_source=github&utm_medium=readme&utm_campaign=reversal-stats) — tick-level Polymarket and Predict.fun crypto Up/Down archives (order books, trades, settlement feeds), free sample days, and a hosted backtester. Results and rounds: CC BY 4.0, attribution "outcometick". Code: MIT.
