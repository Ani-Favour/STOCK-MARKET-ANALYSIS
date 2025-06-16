# STOCK-MARKET-ANALYSIS
This project presents an in-depth SQL-based analysis of historical stock market data from the S&amp;P 500 between 2014 and 2017. Using Microsoft SQL Server, I explored key market metrics, volatility trends, trading volumes, and performance insights across multiple dimensions — including time, stock symbols, and price movement patterns.

- **Which date in the sample saw the largest overall trading volume? On that date, which two stocks were traded most?**
SELECT TOP 3 [date], symbol,
SUM(CAST(volume As Bigint)) as Overall_Trading_Volume
FROM [S&P_500_Stock Prices_2014-2017]
Group by [date], symbol
Order by Overall_Trading_Volume DESC
