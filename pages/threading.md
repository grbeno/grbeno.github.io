## Fetching Data Using Threading

### About the use case

We need to retrieve data using yfinance, but the response time is usually slow. Using a simple loop to fetch information for a large number of tickers doesn’t seem very efficient. A faster approach is needed here, and the threading technique could be a solution. Consider this if the order of the queried symbols does not matter.

First, we check how looping through the data works.

### 1. Using a loop

Looping is a basic solution that works in some cases. The snippet below shows how long the server takes to respond when a loop is applied.

#### Tickers

We also need a ticker list to parse Yahoo Finance data and fetch the beta value. The ticker list can be retrieved from the SEC or from a Wikipedia article on the S&P 500. I choose the first 200 tickers from SEC data.

```python
# 1. Using loop for fetching information

# Get US stock tickers from SEC data
def get_us_sec_tickers() -> list:
    """ Returns a list of US stock tickers from SEC data. """ 
    url = "https://www.sec.gov/files/company_tickers.json"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"
    }
    response = requests.get(url, headers=headers, timeout=10)
    data = response.json()
    result = []
    for i in data:
        result.append(data[str(i)]["ticker"])
    return result
```

Handling internal errors through incomplete exception handling and retrieving the tickers’ beta values. To make the exception handling complete, consider rate-limit errors and other internal issues:

```python
def process_ticker(ticker):
    try:
        stock_data = yf.Ticker(ticker)
        data = stock_data.info["beta"]
        print(f"{ticker}: {data}")
        return data
    except KeyError:
        return None
```

Here is the for loop that parses the first 200 ticker symbols from the SEC endpoint and retrieves their beta values from Yahoo Finance:

```python
LIMIT = 200  # Limit to first N tickers for testing

def main():
    tickers = get_us_sec_tickers()[:LIMIT]  # Limit to first LIMIT tickers for testing
    print(f"Total tickers: {len(tickers)}")
    results = []
    for ticker in tickers:
        data = process_ticker(ticker)
        if data is not None:
            results.append(data)
    print(f"Valid tickers: {len(results)}")
```

Evaluating and printing the elapsed time:

```python 
if __name__ == "__main__":
    # Start time
    start_time = time.time()
    print(f"Start time: {time.strftime('%H:%M:%S', time.localtime(start_time))}")
    # Run the main function
    main()
    # Elapsed time
    elapsed_time = time.time() - start_time
    print(f"Total elapsed time: {elapsed_time:.2f} seconds")
```

Let's see:

```
Start time: 23:19:23
Total tickers: 200
NVDA: 2.375
AAPL: 1.116
GOOGL: 1.112
MSFT: 1.108
...
Valid tickers: 193
Total elapsed time: 87.27 seconds
```

### 2. Threading

Threading is a way to speed up the response time. After clearing the cache to avoid triggering internal yfinance issues, it is important to configure the number of workers (threads) for concurrent processing.

Here’s how it looks.

```python
# 2. Using threads for fetching data

def clear_yfinance_cache() -> None:
    """
    Clear yfinance cache to avoid stale data issues.
    This kind of error: 
    HTTP Error 401: {"finance":{"result":null,"error":{"code":"Unauthorized","description":"Invalid Crumb"}}}
    """
    cache_dir = os.path.join(os.environ["LOCALAPPDATA"], "py-yfinance")
    if os.path.exists(cache_dir):
        shutil.rmtree(cache_dir)

WORKERS = os.cpu_count() * 2  # Number of threads for concurrent processing

def main():
    # Clear stale crumb cache before starting
    clear_yfinance_cache() 
    tickers = get_us_sec_tickers()[:LIMIT]  # Limit to first 200 tickers for testing
    print(f"Total tickers: {len(tickers)}")
    
    results = []
    
    # Use ThreadPoolExecutor for concurrent processing
    with ThreadPoolExecutor(max_workers=WORKERS) as executor:
        futures = [executor.submit(process_ticker, ticker) for ticker in tickers]
        for future in as_completed(futures):
            results.append(future.result())
    
    print(f"Valid tickers: {len([r for r in results if r is not None])}")
```

Run it exactly as in the looping test.

```python
if __name__ == "__main__":
    # Start time
    start_time = time.time()
    print(f"Start time: {time.strftime('%H:%M:%S', time.localtime(start_time))}")
    # Run the main function
    main()
    # Elapsed time
    elapsed_time = time.time() - start_time
    print(f"Total elapsed time: {elapsed_time:.2f} seconds")
```

Let's see:

```
Start time: 23:12:24
Total tickers: 200
MA: 452091084800
AAPL: 3736653529088
JPM: 773751701504
META: 1575052902400
...
Valid tickers: 199
Total elapsed time: 4.79 seconds
```

As you can see, using threading to run the same process on the same data is much faster than using a loop.

In the case of threading, the order will not be the original one, because threads run concurrently and are scheduled independently by the operating system, so execution/completion order is nondeterministic.