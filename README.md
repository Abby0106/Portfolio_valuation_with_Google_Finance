Portfolio Summary Program
Portfolio Summary Program
1. Libraries and Imports
Purpose:
First, we import the necessary libraries:
- tabulate for creating tables.
- dataclasses for creating simple classes for storing data.
- BeautifulSoup from bs4 for parsing HTML data.
- requests for making HTTP requests to fetch data from the web.
```python
!pip install tabulate
from dataclasses import dataclass
from bs4 import BeautifulSoup
from tabulate import tabulate
import requests
```
2. Helper Functions
get_price_by_usd(currency)
- Takes a currency code (e.g., 'CAD' for Canadian Dollars) and fetches its current value in USD.
Portfolio Summary Program
- Constructs a URL to Google Finance to get the exchange rate.
- Uses requests to get the page content and BeautifulSoup to parse the HTML and extract the
exchange rate.
```python
def get_price_by_usd(currency):
 url = f"https://www.google.com/finance/quote/{currency}-USD"
 r = requests.get(url)
 soup = BeautifulSoup(r.content, "html.parser")
 price_fx = soup.find('div', attrs={"data-last-price": True})
 fx = float(price_fx['data-last-price'])
 return fx
```
get_price_info(ticker, exchange)
- Takes a stock ticker and its exchange (e.g., 'MSFT' for Microsoft on NASDAQ) and fetches the
current price and currency.
- Constructs a URL to Google Finance to get the stock's price.
- Uses requests to get the page content and BeautifulSoup to parse the HTML and extract the stock
price and currency.
- If the stock's currency is not USD, it converts the price to USD using get_price_by_usd.
Portfolio Summary Program
```python
def get_price_info(ticker, exchange):
 url = f"https://www.google.com/finance/quote/{ticker}:{exchange}"
 r = requests.get(url)
 soup = BeautifulSoup(r.content, "html.parser")
 price_div = soup.find('div', attrs={"data-last-price": True})
 price = float(price_div["data-last-price"])
 currency = price_div["data-currency-code"]
 usd_price = price
 if currency != 'USD':
 fx = get_price_by_usd(currency)
 usd_price = round(price * fx, 2)

 return {
 "ticker": ticker,
 "exchange": exchange,
 "price": price,
 "currency": currency,
 "usd_price": usd_price
 }
```
3. Classes
@dataclass Stock
Portfolio Summary Program
- Represents a stock.
- Has attributes for the stock ticker, exchange, currency, price, and price in USD.
- When an instance is created, it automatically fetches and sets these values using get_price_info.
```python
@dataclass
class Stock:
 ticker: str
 exchange: str
 currency: str = 'USD'
 price: float = 0
 usd_price: float = 0
 def __post_init__(self):
 price_info = get_price_info(self.ticker, self.exchange)
 if price_info['ticker'] == self.ticker:
 self.price = price_info['price']
 self.currency = price_info['currency']
 self.usd_price = price_info['usd_price']
```
@dataclass Position
Portfolio Summary Program
- Represents a position in a stock portfolio.
- Has attributes for a Stock object and the quantity of shares owned.
```python
@dataclass
class Position:
 stock: Stock
 quantity: int
```
@dataclass Portfolio
- Represents a stock portfolio.
- Has an attribute for a list of Position objects.
- Has a method get_total_value to calculate the total value of the portfolio in USD.
```python
@dataclass
class Portfolio:
 positions: list[Position]

 def get_total_value(self):
 total_value = 0
Portfolio Summary Program
 for position in self.positions:
 total_value += position.quantity * position.stock.usd_price

 return total_value
```
4. Display Function
display_portfolio_summary(portfolio)
- Takes a Portfolio object and displays its summary in a table.
- First checks if the input is a Portfolio object.
- Calculates the total value of the portfolio.
- Creates a list to store data for each position in the portfolio.
- Iterates over each position and adds its data to the list.
- Prints the table using tabulate and prints the total portfolio value.
```python
def display_portfolio_summary(portfolio):
 if not isinstance(portfolio, Portfolio):
 raise TypeError("Please provide an instance of the Portfolio type")
 portfolio_value = portfolio.get_total_value()
 position_data = []
Portfolio Summary Program
 for position in portfolio.positions:
 position_data.append([
 position.stock.ticker,
 position.stock.exchange,
 position.quantity,
 position.stock.usd_price,
 position.quantity * position.stock.usd_price,
 position.quantity * position.stock.usd_price / portfolio_value * 100
 ])

 print(tabulate(position_data,
 headers=["Ticker", "Exchange", "Quantity", "Price",
 "Market Value", "% Allocation"],
 tablefmt='psql',
 floatfmt=".2f"))
 print(f"Total portfolio value: {portfolio_value:,.2f}.")
```
5. Main Block
if __name__ == '__main__':
- Creates some Stock objects.
- Creates a Portfolio object with some Position objects.
Portfolio Summary Program
- Calls display_portfolio_summary to display the portfolio summary.
```python
if __name__ == "__main__":
 shop = Stock('SHOP', 'TSE')
 msft = Stock('MSFT', 'NASDAQ')
 google = Stock('GOOGL', 'NASDAQ')
 portfolio = Portfolio([Position(shop, 10), Position(msft, 50), Position(google, 10)])
 display_portfolio_summary(portfolio)
```
