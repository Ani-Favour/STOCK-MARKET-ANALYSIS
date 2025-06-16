# STOCK-MARKET-ANALYSIS
This project presents an in-depth SQL-based analysis of historical stock market data from the S&amp;P 500 between 2014 and 2017. Using Microsoft SQL Server, I explored key market metrics, volatility trends, trading volumes, and performance insights across multiple dimensions — including time, stock symbols, and price movement patterns.

- **Which date in the sample saw the largest overall trading volume? On that date, which two stocks were traded most?**
```SQL
SELECT TOP 3 [date], symbol,
SUM(CAST(volume As Bigint)) as Overall_Trading_Volume
FROM [S&P_500_Stock Prices_2014-2017]
Group by [date], symbol
Order by Overall_Trading_Volume DESC
```
|date |	symbol |	Overall_Trading_Volume |
|----| ------| -------------------|
|2014-02-24 |	VZ |	618237630 |
|2015-11-17 |	GE |	431332632 |
2016-02-11 |	BAC |	375088650 |

- **On which day of the week does volume tend to be highest? Lowest?**
```SQL
SELECT 
DATENAME(WEEKDAY, [date]) AS Day_of_Week,
AVG(CAST(volume AS BIGINT)) AS Average_Trading_Volume
FROM [S&P_500_Stock Prices_2014-2017]
GROUP BY DATENAME(WEEKDAY, [date])
ORDER BY Average_Trading_Volume DESC
```
|Day_of_Week | Average_Trading_Volume |
|---------|-------------------- |
|Friday | 	4435781 |
|Thursday |	4299304 |
|Wednesday |	4298572 |
|Tuesday |	4188689 |
|Monday |	4031171 |

- **On which date did Amazon (AMZN) see the most volatility, measured by the difference between the high and low price?**
```SQL
 SELECT TOP 1 [date], [high],[low],
([high] - [low]) AS volatility
FROM [S&P_500_Stock Prices_2014-2017]
WHERE symbol = 'AMZN'
ORDER BY volatility DESC
```
|date |	high |	low	volatility |
|---- |----|------------- |
|2017-06-09 |	1012.99 |	927 85.99 |

- **What was the best performing stock in 2015 based on closing price increase?**
```SQL
WITH RankedPrices AS (
    SELECT 
        symbol,
        [close],
        date,
        ROW_NUMBER() OVER (PARTITION BY symbol ORDER BY date ASC) AS rn_asc,
        ROW_NUMBER() OVER (PARTITION BY symbol ORDER BY date DESC) AS rn_desc
    FROM [S&P_500_Stock Prices_2014-2017]
    WHERE date BETWEEN '2015-01-01' AND '2015-12-31')

, FirstLast AS (
    SELECT 
        symbol,
        MAX(CASE WHEN rn_asc = 1 THEN [close] END) AS first_close,
        MAX(CASE WHEN rn_desc = 1 THEN [close] END) AS last_close
    FROM RankedPrices
    GROUP BY symbol)

SELECT 
    symbol,
    ROUND((last_close - first_close) / first_close * 100, 2) AS percent_increase
FROM FirstLast
ORDER BY percent_increase DESC
```
|symbol |	percent_increase |
|------|---------------|
|NFLX |	129.46 |
|AMZN |	119.07 |
|ATVI |	92.3 |
|AYI |	67.14 |
|NVDA |	63.74 |


  
