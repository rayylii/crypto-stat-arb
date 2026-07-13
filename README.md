# crypto-stat-arb

a walk-forward pairs trading backtest on bybit spot crypto using engle-granger cointegration.

## strategy

screen pairs for cointegration on the training window (first 10,000 of 20,000 candles), p < 0.05.

for survivors, fit on train:

p_a = alpha + beta * p_b
spread = p_a - alpha - beta * p_b
z = (spread - mean_train) / std_train

trade the test window (~416 days) with those parameters frozen. enter when z crosses +/-2, exit when abs(z) < 0.5. signal on the previous bar's close, fill on the current bar.

## data

20,000 hourly candles (~2.3 years) per ticker from the bybit v5 kline api, pinned to 2026-07-05, cached to disk. universe is usdt pairs with >$5m turnover at the pinned date, hardcoded sorted list (engle-granger is direction dependent so the order has to be fixed). results are reproducible across runs.

## results

one pair survives the screening: eth|gram, train p = 0.049

out-of-sample: +$244 over 416 days (~19% on ~$1,300 deployed)  
sharpe 0.34  
drawdown -47.3%  

## bugs found during development

lookahead in pair selection, screening on full history instead of train only

same bar execution, signal and fill on the same close

signal/position mismatch, z-score tracked beta-weighted spread but positions were dollar-neutral

no mark to market, drawdown was invisible while positions were open

non-reproducible results from random ticker ordering flipping engle-granger direction

## limitations

survivorship bias in the universe filter (only tickers liquid at the snapshot date were considered). one trade, so sharpe ratio and maximum drawdown are noisy. assumes short positions are executable at the spot price (real-world implementation would require margin borrowing or perpetual futures).

## future work

rolling re-estimation of beta and spread statistics, stop-loss when abs(z) > 4

## stack

python

## run

```bash
git clone https://github.com/rayylii/crypto-stat-arb.git
cd crypto-stat-arb
pip install -r requirements.txt
jupyter notebook analysis.ipynb
```