# Polymarket 5-minute Up/Down: how often does a 90–99% favourite lose?

[中文](README.zh-CN.md) · Data: [outcometick](https://outcometick.com?utm_source=github&utm_medium=readme&utm_campaign=reversal-stats) · Sister repo: [Predict.fun reversal stats](https://github.com/outcometick/predict-fun-reversal-stats)

In Polymarket's 5-minute crypto Up/Down markets, once one side's price first reaches 90%, 93%, 95%, 98% or 99%, how often does that side still lose at the close? This repository answers that from tick-level order-book data and **updates itself every day**.

**Coverage:** 2026-09-02 → 2026-11-19 (UTC), 79 days, 159,067 settled 5-minute markets across BTC, BNB, DOGE, ETH, HYPE, SOL, XRP. Last updated 2026-11-19.

## Headline — all assets, mid price

| favourite first reached | rounds that got there | favourite lost | reversal rate | 95% interval | avg price at trigger | win rate − price |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| 90% | 157,327 | 14,445 | 9.18% | 9.04% – 9.32% | 0.9031 | 0.50 pp |
| 93% | 156,594 | 10,028 | 6.40% | 6.28% – 6.53% | 0.9319 | 0.41 pp |
| 95% | 155,069 | 6,874 | 4.43% | 4.33% – 4.54% | 0.9515 | 0.41 pp |
| 98% | 137,031 | 2,098 | 1.53% | 1.47% – 1.60% | 0.9808 | 0.39 pp |
| 99% | 15,320 | 64 | 0.42% | 0.33% – 0.53% | 0.9940 | 0.18 pp |

![realised vs implied reversal rate](charts/calibration.svg)

*Implied* is 1 − the average price paid at the trigger. When the realised rate sits above it, the favourite at that price lost more often than its price said; "win rate − price" is the same gap seen from the buyer's side, before fees.

## By asset — reversal rate at each threshold (mid)

| asset | 90% | 93% | 95% | 98% | 99% | rounds at 95% |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| **all** | 9.18% | 6.40% | 4.43% | 1.53% | 0.42% | 155,069 |
| BTC | 8.37% | 5.86% | 4.13% | 1.42% | 0.41% | 22,560 |
| BNB | 9.60% | 6.67% | 4.69% | 1.56% | 0.49% | 22,439 |
| DOGE | 9.54% | 6.68% | 4.55% | 1.61% | 0.44% | 22,411 |
| ETH | 8.61% | 5.87% | 3.96% | 1.48% | 0.74% | 22,552 |
| HYPE | 9.19% | 6.50% | 4.65% | 1.57% | 0.18% | 20,027 |
| SOL | 9.55% | 6.49% | 4.49% | 1.56% | 0.49% | 22,543 |
| XRP | 9.42% | 6.77% | 4.58% | 1.56% | 0.27% | 22,537 |

## By price basis — all assets

| threshold | mid | best bid | trade |
| --- | ---: | ---: | ---: |
| 90% | 9.18% | 8.29% | 7.53% |
| 93% | 6.40% | 5.57% | 5.37% |
| 95% | 4.43% | 3.69% | 4.05% |
| 98% | 1.53% | 1.15% | 2.04% |
| 99% | 0.42% | 0.57% | 1.29% |

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
