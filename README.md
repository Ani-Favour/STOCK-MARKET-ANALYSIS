# STOCK-MARKET-ANALYSIS
This project presents an in-depth SQL-based analysis of historical stock market data from the S&amp;P 500 between 2014 and 2017. Using Microsoft SQL Server, I explored key market metrics, volatility trends, trading volumes, and performance insights across multiple dimensions — including time, stock symbols, and price movement patterns.


## Contents/ Key Analyses Performed
 -  [Market Volume Insights](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS#market-volume-insights)
 -  [Volatility Analysis](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS#volatility-analysis)
 -  [Performance Metrics](https://github.com/Ani-Favour/STOCK-MARKET-ANALYSIS?tab=readme-ov-file#performance-metrics)
    
## Market Volume Insights
- **Identified dates with the highest overall trading volume.**
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

- **Ranked days of the week by average volume.**
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

- **Determined stocks with the highest average daily trading volumes.?**
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
- **Detected the most volatile trading days across the market.**
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


- **Flagged high-volatility stock days (e.g., >5% intraday swings).**
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

- **Calculated a custom Volatility Index by day.**
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
- **Found best performing stocks in 2015 by % price increase.**
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

- **What was the average daily return (close - open) for each stock?**
```SQL
SELECT symbol,
ROUND(AVG([close]-[open]), 2) as AVG_Daily_Return
FROM [S&P_500_Stock Prices_2014-2017]
Group by symbol
Order by avg_daily_return DESC
```
|symbol |	AVG_Daily_Return |
|------ |------------- |
|EQIX |	0.21 |
|SHW |	0.17 |
|MTD |	0.16 |
|HUM |	0.14 |
|TDG |	0.14 |


- **How did the top 5 most traded stocks in 2014 perform by the end of 2017?**
```SQL
SELECT TOP 5 symbol,
ROUND(SUM(CAST(volume AS BIGINT)), 2) AS Total_Volume
FROM [S&P_500_Stock Prices_2014-2017]
WHERE date >= '2014-01-01' AND date <= '2014-12-31'
GROUP BY symbol
ORDER BY Total_Volume DESC


SELECT symbol,
MIN(CASE WHEN YEAR(date) = 2014 THEN [close] END) AS Close_2014,
MAX(CASE WHEN YEAR(date) = 2017 THEN [close] END) AS Close_2017
FROM [S&P_500_Stock Prices_2014-2017]
Where symbol IN ('BAC', 'AAPL', 'FB', 'MSFT', 'CSCO')
GROUP BY symbol
```
|symbol |	Close_2014 |	Close_2017 |
|------- | --------- | --------- |
|MSFT |	34.98 |	86.85 |
|AAPL |	71.4 |	176.42 |
|BAC |	14.51 |	29.88 |
|CSCO |	21.35 |	38.74 |
|FB |	53.53 |	183.03 |


- **Computed stocks daily returns and highlighted biggest gain/loss days:**
- largest single-day price increase (close - open)
```SQL 
SELECT symbol, [date], [open], [close],
ROUND([close] - [open], 2) AS price_increase
FROM [S&P_500_Stock Prices_2014-2017]
ORDER BY price_increase DESC
```
|symbol |	date |	open |	close |	price_increase |
|------ | ---- | ---- |-----| ------------- |
|PCLN	|2014-05-27	|1206.16003417969	|1259.10998535156	|52.95
|PCLN	|2014-08-04	|1247.55004882813	|1299.92004394531	|52.37
|PCLN	|2014-04-01	|1203.2099609375	|1251.36999511719	|48.16
|PCLN	|2016-06-06	|1302.52001953125	|1349.09997558594	|46.58
|PCLN	|2017-11-09	|1656.93005371094	|1703.14001464844	|46.21
|PCLN	|2016-02-19	|1238.28002929688	|1283.73999023438	|45.46
|PCLN	|2017-07-07	|1873.85998535156	|1918.5	                |44.64
|PCLN	|2014-11-11	|1123.09997558594	|1167.35998535156	|44.26
|PCLN	|2016-11-07	|1437.44995117188	|1480.32995605469	|42.88
|AMZN	|2017-10-27	|1058.14001464844	|1100.94995117188	|42.81


- negative daily return
```SQL
SELECT symbol,[date],
ROUND([close] - [open], 2) AS negative_daily_return
FROM [S&P_500_Stock Prices_2014-2017]
WHERE [open] IS NOT NULL AND [close] IS NOT NULL
ORDER BY negative_daily_return ASC
```
|symbol |	date |	negative_daily_return |
|------ |---- |------------------- |
|PCLN |2017-11-07 |-100.98 |
|CMG |2015-11-20 |-75.81 |
|PCLN |2014-04-04 |-69.92 |
|CMG |2014-04-17 |-63.09 |
|PCLN |2016-06-24 |-61.64 |



## Temporal Patterns:
- **Which day of the week sees the highest average close prices across the market?**
```SQL
SELECT 
DATENAME(WEEKDAY, [date]) AS day_of_week,
AVG([close]) AS avg_close_price
FROM [S&P_500_Stock Prices_2014-2017]
GROUP BY DATENAME(WEEKDAY, date)
ORDER BY avg_close_price DESC
```
|day_of_week |	avg_close_price |
|--------- |------------- |
|Friday|	86.51 |
|Wednesday|	86.42 |
|Thursday|	86.32 |
|Monday |	86.32 |
|Tuesday |	86.27 |

- **How do monthly average closing prices for a stock trend over time?**
 ```SQL
Alter table [S&P_500_Stock Prices_2014-2017]
 Add [Month] Varchar(50)

 Update [S&P_500_Stock Prices_2014-2017]
 Set [Month] = Datename(Month, [date])


SELECT [Month],
AVG([close]) AS avg_close_price
FROM [S&P_500_Stock Prices_2014-2017]
WHERE symbol = 'ABBV'
GROUP BY [Month]
ORDER BY 
    Case [Month]	
		WHEN 'January' THEN 1
        WHEN 'February' THEN 2
        WHEN 'March' THEN 3
        WHEN 'April' THEN 4
        WHEN 'May' THEN 5
        WHEN 'June' THEN 6
        WHEN 'July' THEN 7
        WHEN 'August' THEN 8
        WHEN 'September' THEN 9
        WHEN 'October' THEN 10
        WHEN 'November' THEN 11
        WHEN 'December' THEN 12
    END
```
|Month |	avg_close_price |
|----- | ------------- |
|January |	57.89 |
|February |	56.12 |
|March |	58.11 |
|April |	59.03 |
|May |	61.56 |
|June |	63.70 |
|July |	65.03 |
|August |	64.93 |
|September |	66.06 |
|October |	66.23 |
|November |	70.47 |
|December |	70.22 |

## Pattern Recognition:
- **Identified bullish trading patterns (close > open)**
```SQL
SELECT symbol,
COUNT(*) AS Bullish_Days
FROM [S&P_500_Stock Prices_2014-2017]
WHERE [close] > [open]
GROUP BY symbol
ORDER BY Bullish_Days DESC
```
|symbol |	Bullish_Days |
|------ | ----------- |
|INTU	|578 |
|ADBE	|577 |
|RMD	|571 |
|SNPS	|568 |
|IDXX	|563 |








  
