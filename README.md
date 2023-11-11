# StatisticalArbitrageCrypto
Trying some statistical arbitrage opportunity detection algorithm using CCXT and Python.


# Exploring Statistical Arbitrage in Cryptocurrency Markets Using Python and CCXT


## Introduction

Statistical arbitrage is a strategy that seeks to exploit pricing inefficiencies between different markets or assets. In the cryptocurrency world, these inefficiencies can be quite prevalent due to the fragmentation of markets across various exchanges. This blog will guide you through the process of using Python and the CCXT library to detect potential arbitrage opportunities across different cryptocurrency exchanges.

## Setting Up the Environment

To begin, ensure that you have Python installed on your system. We'll be using the CCXT library, which is a powerful tool to connect and trade with cryptocurrency exchanges and payment processing services worldwide. You can install it using pip:

```bash
pip install ccxt
```

## Selecting Exchanges and Cryptocurrency Pairs

CCXT provides access to a plethora of exchanges, but not all exchanges are created equal. Some support a wide variety of functions, while others are more limited. For our arbitrage detection, we need exchanges that support the `fetchOHLCV` function, which allows us to fetch Open/High/Low/Close/Volume data.

Here's a snippet to filter and select two random exchanges that support this functionality:

```python
import ccxt
import random

# Get a list of all exchanges that support fetchOHLCV
exchanges_with_ohlcv = [exchange for exchange in ccxt.exchanges if getattr(ccxt, exchange)().has['fetchOHLCV']]

# Select two random exchanges from this list
selected_exchanges = random.sample(exchanges_with_ohlcv, 2)

# Initialize the selected exchanges
exchange1 = getattr(ccxt, selected_exchanges[0])({'enableRateLimit': True})
exchange2 = getattr(ccxt, selected_exchanges[1])({'enableRateLimit': True})
```

## Fetching Data

Next, we'll need to find common symbols that both exchanges support. We load the markets and identify the symbols that are available on both platforms. Then, we'll fetch the OHLCV data for these symbols.

```python
def fetch_common_symbols(exchange1, exchange2):
    # Load markets for both exchanges
    exchange1.load_markets()
    exchange2.load_markets()

    # Find common symbols
    common_symbols = list(set(exchange1.symbols) & set(exchange2.symbols))
    if not common_symbols:
        raise Exception("No common symbols found.")
    
    return common_symbols

# Fetch OHLCV data for a common symbol
def fetch_ohlcv(exchange, symbol):
    ohlcv = exchange.fetch_ohlcv(symbol, '5m', since=exchange.milliseconds() - (24 * 60 * 60 * 1000))
    return ohlcv
```

## Calculating Arbitrage Opportunities

With our data in hand, we can now look for discrepancies in pricing between the two selected exchanges. We will assume that we identify an opportunity based on the opening price but execute it at the closing price to simulate a delay in execution.

Here's a function to calculate potential profits considering transaction fees and estimated slippage:

```python
def calculate_arbitrage_profit(open_price1, close_price1, open_price2, close_price2, volume, transaction_fee1, transaction_fee2, slippage):
    # Buy on the exchange with the lower open price and sell on the exchange with the higher open price
    if open_price1 < open_price2:
        buy_cost = volume / open_price1 * (1 + transaction_fee1 + slippage)
        sell_revenue = volume / close_price2 * (1 - transaction_fee2)
    else:
        buy_cost = volume / open_price2 * (1 + transaction_fee2 + slippage)
        sell_revenue = volume / close_price1 * (1 - transaction_fee1)
    
    # Calculate profit
    profit = sell_revenue - buy_cost
    return profit
```

## Conclusion

This blog has provided a high-level overview of setting up a Python environment to detect statistical arbitrage opportunities in cryptocurrency markets. We have covered selecting exchanges, fetching data, and calculating potential profits. Keep in mind that real-world application would require a more robust setup, including real-time data, better slippage estimation, and a thorough risk management strategy.

Happy trading!
