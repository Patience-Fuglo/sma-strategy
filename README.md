# Building and Deploying a Stock Trading Bot in Python: Simple Moving Average (SMA)

This project demonstrates how to build a simple algorithmic stock trading bot in Python using the **Simple Moving Average (SMA)** strategy. The bot trades Apple Inc. (AAPL) stock by buying when the price is above the 50-day SMA and holding or selling otherwise. It includes backtesting against SPY and live paper trading via Alpaca API.

---

## Table of Contents

1. [Setting up the Environment](#setting-up-the-environment)
2. [Importing Libraries and Fetching Data](#importing-libraries-and-fetching-data)
3. [Calculating the 50-day SMA](#calculating-the-50-day-sma)
4. [Implementing the Trading Strategy](#implementing-the-trading-strategy)
5. [Backtesting the Strategy and Comparing with SPY](#backtesting-the-strategy-and-comparing-with-spy)
6. [Plotting the Results](#plotting-the-results)
7. [Connecting to Alpaca for Live Trading](#connecting-to-alpaca-for-live-trading)
8. [Implementing Live Trading Algorithm](#implementing-live-trading-algorithm)
9. [Running the Algorithm](#running-the-algorithm)

---

## Setting up the Environment

Install the required Python libraries:

```bash
pip install pandas numpy matplotlib yfinance alpaca-trade-api
```

---

## Importing Libraries and Fetching Data

Fetch historical stock data using `yfinance`:

```python
import numpy as np
import pandas as pd
import yfinance as yf
import matplotlib.pyplot as plt

symbol = 'AAPL'
start_date = '2015-01-01'
end_date = '2022-12-31'
data = yf.download(symbol, start=start_date, end=end_date)
```

---

## Calculating the 50-day SMA

Calculate the 50-day Simple Moving Average:

```python
data['SMA_50'] = data['Close'].rolling(window=50).mean()
```

---

## Implementing the Trading Strategy

Create trading signals based on SMA crossover:

```python
data['Signal'] = np.where(data['Close'] > data['SMA_50'], 1, 0)
```

---

## Backtesting the Strategy and Comparing with SPY

Calculate daily and cumulative returns to evaluate performance:

```python
data['Daily_Return'] = data['Close'].pct_change()
data['Strategy_Return'] = data['Daily_Return'] * data['Signal'].shift(1)
data['Cumulative_Return'] = (1 + data['Strategy_Return']).cumprod()

spy_data = yf.download('SPY', start=start_date, end=end_date)
spy_data['Daily_Return'] = spy_data['Close'].pct_change()
spy_data['Cumulative_Return'] = (1 + spy_data['Daily_Return']).cumprod()
```

---

## Plotting the Results

Visualize cumulative returns of your strategy vs. SPY benchmark:

```python
plt.figure(figsize=(12, 6))
plt.plot(data.index, data['Cumulative_Return'], label='SMA Strategy')
plt.plot(spy_data.index, spy_data['Cumulative_Return'], label='SPY')
plt.xlabel('Date')
plt.ylabel('Cumulative Returns')
plt.legend()
plt.show()
```

---

## Connecting to Alpaca for Live Trading

Set up the Alpaca API to enable live paper trading:

```python
from alpaca_trade_api import REST

api_key = 'YOUR_API_KEY'
api_secret = 'YOUR_SECRET_KEY'
base_url = 'https://paper-api.alpaca.markets'  # Paper trading URL

api = REST(api_key, api_secret, base_url)
```

---

## Implementing Live Trading Algorithm

Define functions to check positions and place trades based on live prices and SMA:

```python
import time

def check_positions(symbol):
    positions = api.list_positions()
    for position in positions:
        if position.symbol == symbol:
            return int(position.qty)
    return 0

def trade(symbol, qty):
    current_price = api.get_latest_trade(symbol).price
    historical_data = yf.download(symbol, start=start_date, end=end_date, interval='1d')
    historical_data['SMA_50'] = historical_data['Close'].rolling(window=50).mean()

    if current_price > historical_data['SMA_50'][-1]:
        if check_positions(symbol) == 0:
            api.submit_order(
                symbol=symbol,
                qty=qty,
                side='buy',
                type='market',
                time_in_force='gtc'
            )
            print(f"Buy order placed for {symbol}")
        else:
            print(f"Holding {symbol}")
```

---

## Running the Algorithm

Run the live trading loop with a daily delay:

```python
symbol = 'AAPL'
qty = 10

while True:
    trade(symbol, qty)
    time.sleep(86400)  # Sleep for 1 day (86400 seconds)
```

---

## Notes

* Use your own Alpaca API keys and keep them secure.
* This strategy is a simple example; real trading requires deeper testing and risk management.
* You can adapt the code for other stocks or assets.
* Consider version control with GitHub to track improvements over time.

