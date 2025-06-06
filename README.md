# black-scholes-engine
Python-based options pricing models, analytics, stochastic options models, European options pricing, 

## Using YFinance (version1)

## Black Scholes Engine 
Option Pricing Engine with Greeks (Analytical + Monte Carlo)

## Stack
Language: Python 3.10+, 
Tools: NumPy, SciPy, Matplotlib, pandas, Jupyter Notebook, pytest

## Types of Data Required
| **Type of Data**              | **Purpose**                                     | **Example**      |
| ----------------------------- | ----------------------------------------------- | ---------------- |
| **Stock Prices (historical)** | To estimate **realized/historical volatility**  | AAPL, TSLA, etc. |
| **Option Chain (live data)**  | For **validation** of your pricing model        | Call/put prices  |
| **Risk-Free Rate**            | Used in Black-Scholes model (e.g., 1Y Treasury) | 5.00%            |
| **Dividend Yield**            | If stock pays dividends (optional for European) | 0.70%            |

## Where to Get the Data
| **Source**                                   | **Data Type**                     | **How to Access**         |
| -------------------------------------------- | --------------------------------- | ------------------------- |
| [Yahoo Finance](https://finance.yahoo.com)   | Historical stock prices + options | `yfinance` Python package |
| [FRED](https://fred.stlouisfed.org)          | Risk-free rates (e.g., 1Y T-bill) | `fredapi`                 |
| [Quandl](https://www.quandl.com)             | Premium/Free economic data        | `quandl` API              |
| [Polygon.io](https://polygon.io)             | Real-time market data (paid)      | REST API                  |
| [Alpha Vantage](https://www.alphavantage.co) | Historical + options data         | Free API key              |

## How to Clean the Data 
A. Stock Price Data (for Volatility)
import yfinance as yf

ticker = yf.Ticker("AAPL")
hist = ticker.history(period="1y")  # Get 1 year of daily data

# Clean and inspect
hist = hist[['Close']].dropna()
hist.columns = ['price']
hist.head()

B. Estimate Volatility 
Create annualized historical volatility 
import numpy as np

hist['log_ret'] = np.log(hist['price'] / hist['price'].shift(1))
vol_daily = hist['log_ret'].std()
vol_annual = vol_daily * np.sqrt(252)

C. Risk-Free Rate (from FRED)
from fredapi import Fred
fred = Fred(api_key='your_api_key_here')

# Get 1-year Treasury constant maturity rate
rate = fred.get_series('GS1')  # Can also use GS3M for 3-month T-bill
rate = rate[-1] / 100  # Convert to decimal (e.g., 5% -> 0.05)

## Integrate the Engine
from src.black_scholes import black_scholes_price

S = hist['price'].iloc[-1]     # Current stock price
K = 150                        # Strike price
T = 30 / 365                   # Time to maturity in years
r = rate                      # Risk-free rate from FRED
sigma = vol_annual             # Vol from Yahoo data

price = black_scholes_price(S, K, T, r, sigma, option_type='call')
print(f"Call Option Price: ${price:.2f}")

## Bonus Tips
Data cleaning checklist: drop rows with missing prices, ensure proper datetimeindex, check for splits/dividends and adjust if needed, remove outlier (e.g. returns > +-20% daily)

Backtest the engine: compare your prices vs real option chain prices from 'yfinance', track pricing error: ' abs(your_price - market_price) ' 
