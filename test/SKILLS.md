---
name: stock-news-backtest
description: Use this skill when the user wants to research stock news, collect market news or sentiment data, combine it with historical price data, and design or run simple stock backtests. Trigger for requests involving stock news APIs, Alpha Vantage News & Sentiment, Finnhub company news, yfinance price data, pandas backtesting, event-driven backtests, sentiment thresholds, holding-period tests, or evaluating whether news signals have predictive value.
---

# Stock News Backtest

Use this skill to help the user build and improve workflows for stock-news-driven backtesting.

## Core workflow

1. Clarify ticker universe, market, date range, and whether the user wants US stocks, HK stocks, A-shares, crypto, or ETFs.
2. Prefer this MVP stack:
   - `Alpha Vantage NEWS_SENTIMENT` or Finnhub company news for news data.
   - `yfinance` for historical OHLCV price data.
   - `pandas` and `numpy` for signal generation and event-driven backtesting.
   - `matplotlib` for basic charts.
3. Start with simple event studies before complex ML:
   - Filter news by ticker.
   - Convert publication timestamp to the next valid trading day.
   - Define signal rules such as sentiment score > 0.25 or negative score < -0.25.
   - Enter on next trading day open or close.
   - Hold for 1, 3, 5, 10, and 20 trading days.
   - Measure average return, median return, win rate, volatility, max drawdown, and benchmark-relative return.
4. Always warn about common backtest traps:
   - look-ahead bias
   - survivorship bias
   - missing delisted stocks
   - split/dividend adjustment
   - news timestamp and timezone mismatch
   - duplicate articles
   - API rate limits
   - transaction costs and slippage
   - overfitting thresholds

## Recommended commands

For a basic Python environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pandas numpy yfinance requests matplotlib
