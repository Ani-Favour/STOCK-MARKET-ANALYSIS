# STOCK-MARKET-ANALYSIS
This project presents an in-depth SQL-based analysis of historical stock market data from the S&amp;P 500 between 2014 and 2017. Using Microsoft SQL Server, I explored key market metrics, volatility trends, trading volumes, and performance insights across multiple dimensions — including time, stock symbols, and price movement patterns.


## Contents/ Key Analyses Performed
 -  [Market Volume Insights](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS#market-volume-insights)
 -  [Volatility Analysis](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS#volatility-analysis)
 -  [Performance Metrics](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS?tab=readme-ov-file#performance-metrics)
    
## Market Volume Insights
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

- **Which stock had the highest average daily trading volume?**
```SQL
SELECT symbol,
AVG(CAST(volume AS BIGINT)) AS Average_Daily_Trading_Volume
FROM [S&P_500_Stock Prices_2014-2017]
GROUP BY symbol
ORDER BY Average_Daily_Trading_Volume DESC
```
|symbol |	Average_Daily_Trading_Volume |
|------ | -------------------- |
|BAC | 	89362903 |
|AAPL |	45169571 |
|GE |	41443942 |
|AMD |	33289509 |
|F | 32914300 |


## Volatility Analysis
- **On which date did the overall market experience the most volatility?**
```SQL
SELECT [date],
ROUND(SUM([high] - [low]), 2) AS total_market_volatility
FROM [S&P_500_Stock Prices_2014-2017]
WHERE [high] IS NOT NULL AND [low] IS NOT NULL
GROUP BY [date]
ORDER BY total_market_volatility DESC
```
|date |	total_market_volatility |
|---- | --------------------- |
|2015-08-24 |	3361.53 |
|2015-08-25 |	1893.18 |
|2016-11-09 |	1853.15 |
|2016-01-20 |	1726.83 |
|2016-01-13 |	1677.01 |


- **Which symbol had the highest number of high-volatility days (e.g., when (high - low) > 5%)?**
```SQL
SELECT symbol,
COUNT(*) AS high_volatility_days
FROM [S&P_500_Stock Prices_2014-2017]
WHERE [high] IS NOT NULL AND [low] IS NOT NULL AND [open] IS NOT NULL
  AND (([high] - [low]) / [open]) * 100 > 5
GROUP BY symbol
ORDER BY high_volatility_days DESC
```
|symbol |	high_volatility_days |
|------ |----------------- |
|CHK |	391 |
|AMD |	286 |
|INCY |	254 |
|FCX |	245 |
|RRC |	215 |

- **Create a volatility index across all stocks — which days score highest?**
```SQL
SELECT [date],
ROUND(AVG(((high - low) / NULLIF([open], 0)) * 100), 2) AS Volatility_Index
FROM [S&P_500_Stock Prices_2014-2017]
WHERE [high] IS NOT NULL AND [low] IS NOT NULL AND [open] IS NOT NULL
GROUP BY [date]
ORDER BY Volatility_Index DESC
```
|date |	Volatility_Index |
|----|--------------- |
|2015-08-24 |	9.06 |
|2015-08-25 |	4.91 |
|2016-01-20 |	4.84 |
|2016-11-09 |	4.56 |
|2016-01-13 |	4.35 |

## Performance Metrics
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
|------| --------------- |
|NFLX |	129.46 |
|AMZN |	119.07 |
|ATVI |	92.3 |
|AYI |	67.14 |
|NVDA |	63.74 |


















  
