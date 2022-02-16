# Backtesting and Optimising Your Own Trading Strategy
## Introduction
Backtesting is the process of seeing how well your trading strategy would have done using historical data. It allows you to assess whether your strategy would perform well in the future, with the assumption that what worked well in the past is also likely to work well in the future. Optimisation is to adjust variables that are used to form your trading strategy and look for the best mix of variables that would achieve the highest result, with a metric that you have chosen beforehand.

Backtesting and optimisation of trading strategy are more commonly done by institutional investors than by retail investors. Nonetheless, they are equally if not more useful for retail investors as well because retail investors are often affected by their own behavioural biases. They may form their trading strategies based on piecemeal information and do not fully understand how well their strategies have done. They are also more likely to make their trading decisions on impulse and lack discipline to stick with their trading strategy. With backtesting, they would have a clearer picture of which of their strategies are more feasible and more likely to follow through with their strategies.

In this article, I will introduce how you can backtest and optimize your own trading strategy using Python.

## Data Preparation
Before I discuss how we would perform backtesting and optimisation in Python, I first outline how I obtain my stock price data. I use the historical daily stock price data of Tesla in the past two years for illustrating purpose only. I obtain stock price data using the Yahoo Finance API. The data output extracted from the Yahoo Finance API is in json format. l extract the timestamp, Open, Close, High and Low price data and save them in a pandas dataframe. I then calculate the daily return as a percentage change between two consecutive closing prices.

To visualise Tesla’s stock price data over the past two years, I plot a candlestick chart. In the below chart, we can see that over the course of the past two years, the price of Tesla has been quite volatile ranging from around $600 to $1200 but it generally follows an upward trend.

![candlestick](https://github.com/tonytsoi/backtesting/blob/main/assets/candlestick.JPG?raw=true)

## Backtesting the Moving Average Strategy
To illustrate how to backtest a trading strategy, I apply a common trading strategy, the moving average strategy, to Tesla’s historical stock price. A moving average is the average calculated over a moving window of a certain length of period which would smooth out the price data. For example, a 5-day simple moving average (“**SMA**”) calculates the average daily closing prices over the previous 5 days for each day.

Another type of moving average is the exponential moving average (“**EMA**”) which places a greater emphasis on the more recent data. Hence, the EMA is more significantly affected by recent prices changes than the SMA. I set out below the formula for the EMA.

<img src="https://render.githubusercontent.com/render/math?math=\Large \begin{matrix} EMA_{t} = Price_{t} \times \frac{2}{1+N} + EMA_{t-1} \times (1-\frac{2}{1+N})\\N=no.\:of\:days \end{matrix}">

Instead of calculating the EMA of the closing price using the above formula, I use the pre-defined function in the “TA-Lib” library. The TA-Lib library is a useful library to perform technical analysis in Python. I compute the 5-day, 20-day and 50-day EMA of the closing price.

![EMA](https://github.com/tonytsoi/backtesting/blob/main/assets/EMA.jpg?raw=true)

One type of trading strategy that applies moving averages is to take the crossovers of two moving averages as buy or sell signals. For example, when a shorter-term moving average crosses from below to above a longer-term moving average, it indicates an upward trend and is considered as a buy signal. Whereas when the shorter-term moving average crosses from above to below the longer-term moving average, it indicates a downward trend and is considered as a sell signal.

I set out the buying conditions of my moving average strategy as follows:

**a.** when the 5-day EMA crosses from below to above the 20-day EMA; and

**b.** when the closing price, 5-day EMA and 20-EMA are above the 50-EMA.

The selling conditions would be the opposite of the above buying conditions.

To apply the moving average strategy to Tesla’s historical daily closing price, I first create a column named “Position” in which the value 1 represents buying or holding the stock and 0 represents selling or not holding the stock. I take the signal from the previous day and trade in the following day which means that I would have to shift the “Position” column one day forward. I then calculate my daily return of the strategy by multiplying the daily return with the position of the day. Finally, I compute the cumulative return by calculating the compound return of the moving average strategy. I also compute the cumulative return for a strategy which I buy the stock at the start of the period and hold it thereafter i.e. a buy-and-hold strategy.

We can see in the below chart that the moving average strategy performed slightly better than the buy-and-hold strategy in the first year, and was then surpassed in the later year.

![moving average](https://github.com/tonytsoi/backtesting/blob/main/assets/EMA_strategy.jpg?raw=true)

One might be tempted to jump to the conclusion that the buy-and-hold strategy is a better strategy than the moving average strategy. However, it would be more appropriate to assess the risk-return tradeoff of the strategies. A common metric to measure the risk-return tradeoff is the Sharpe Ratio, which is calculated by subtracting the risk-free rate from the mean return and divided by the standard deviation of the return.

<img src="https://render.githubusercontent.com/render/math?math=\Large Sharpe \: Ratio=\frac{R_{p}-R_{f}}{\sigma _{p}}">

In practice, the rate of the U.S. Treasury Bond or the return of the safest investment that the investor could have obtained would be used as the risk-free rate. For simplicity, I assume the annual risk-free rate to be at 1%. To annualise the Sharpe ratio, we multiply it by the square root of the total number of trading days per year which I assume to be 252 days.

The annualised Sharpe Ratio of the buy-and-hold strategy is 1.59 whereas that of the moving average strategy is at 1.83. In other words, the moving average strategy have outperformed the buy-and-hold strategy on a risk-return tradeoff perspective.

## Backtesting and Optimisting the Strategy based on Candlestick

In this subsection, I introduce a trading strategy based on the candlestick which also involves a parameter that would need to be optimised. First, I split the data into a training and test set in which roughly the first 80% of the data points would fall into the training set and the rest in the test set.

A candlestick displays the high, low, open and closing stock price for a certain length of time. It reflects the relative strength of the buyers and the sellers. For example, a bullish candlestick shows that the buyers are in a relatively stronger position, which push the prices upward and the stock closes at a higher price. A bearish candlestick shows that the sellers are in a stronger position which lead to the stock to close at a lower price.

![candlestick pattern](https://github.com/tonytsoi/backtesting/blob/main/assets/candlestick%20pattern1.jpg?raw=true)

To form a simple trading strategy from the candlestick, I only focus on the relative position of the closing price to the high and low price. I calculate the ratio of the difference in the closing price and the low price over the length of the candlestick (the “To form a simple trading strategy from the candlestick, I only focus on the relative position of the closing price to the high and low price. I calculate the ratio of the difference in the closing price and the low price over the length of the candlestick (the “CLHL ratio”).”).

<img src="https://render.githubusercontent.com/render/math?math=\Large CLHL\: Ratio=\frac{Price_{Close}-Price_{Low}}{Price_{High}-Price_{Low}}">

I then form my strategy based on the CLHL ratio in which I buy when the CLHL ratio is greater than a particular threshold and sell when the ratio is smaller than the threshold. To determine the optimal level of the CLHL ratio threshold, I use the Sharpe Ratio as the metric to measure the performance of the strategy i.e. the optimal level of the threshold is when the Sharpe Ratio is the highest.

For the optimisation of the candlestick strategy, first I define the optimisation function that takes the input of the data, the start and end point and the step. The threshold would increase by the amount of the step in each loop, where I would calculate the Sharpe Ratio of the strategy. I choose the level of the step to be at 0.01 i.e. for each loop the threshold would increase by 0.01. After I calculate the Sharpe Ratios for the strategy at every level of the threshold, I sort them in descending order to obtain the level of the threshold that would yield the highest Sharpe Ratio.

Optimising the strategy over the training set gives a result of the optimal threshold at 0.48. I save this result in a variable named threshold to be used when I apply the strategy to the test set.

I then apply the candlestick strategy to the test set using the optimal threshold from the training set. This function is more or less the same as the optimisation function above, which takes the data and the optimal threshold as inputs and returns the Sharpe Ratio of the strategy as the output.

The annualised Sharpe Ratios of the strategy at the CLHL ratio threshold of 0.48 over the training set and the test set are 1.74 and 1.98 respectively. It suggests that the strategy has performed even better over the test set than the training set. If we compute the annualised Sharpe Ratios of the buy-and-hold strategy over the training set and the test set, it gives a result of 1.69 and 1.10 respectively. This suggests that the candlestick strategy has outperformed the buy-and-hold strategy in both the training set and the test set.

I set out the cumulative return of both strategies over time for the training set and the test set in the below charts. For the training set, although the Sharpe Ratio is higher for the candlestick strategy, the buy-and-hold strategy generate a higher cumulative return at the end of the period. For the test set, it shows that the candlestick strategy would offer a higher cumulative return.

![training set](https://github.com/tonytsoi/backtesting/blob/main/assets/training%20set.jpg?raw=true)
![test set](https://github.com/tonytsoi/backtesting/blob/main/assets/test%20set.jpg?raw=true)

I also apply the candlestick strategy over the full length of the 2-year data. The Sharpe Ratio of the candlestick strategy is at 1.78 which remains to be higher than the buy-and-hold strategy of 1.59. However, the cumulative return plot shows that the overall cumulative return of the buy-and-hold strategy is higher than the candlestick strategy.

![full history](https://github.com/tonytsoi/backtesting/blob/main/assets/full%20history.jpg?raw=true)

## Limitations
Technical analysis assumes that what worked well in the past is likely to work well in the future too. However, this might not be the case as there could be structural changes, for example, how market participants react to the prices changes might vary. Furthermore, one strategy might work for a particular security but it does not mean that it would work for other securities as well.

After all, backtesting only shows that one strategy works well for a particular security in a specific period. There is no inference or guarantee that the strategy can generalise well without further research to back that up.

## Adding More Complexities
In the above illustrations, I only use some simple strategies to show how we can backtest and optimise strategies. A more advanced trading strategy might look at various features at the same time such as trend, support and resistance levels etc. together with the candlestick patterns.

Other features can be included such as stop-loss orders, or the effect of transaction costs which is particularly important to day-trade strategies as you may want to limit the number of transactions to reduce the transaction costs.

Backtesting can also be used in strategies that are based on fundamental analysis, or in fact any strategies that can be quantified. However, the limitations discussed above would still apply in these other strategies.

## Conclusion
Backtesting and optimisation of trading strategies provide a clearer picture to investors to understand how their strategies have achieved and to enhance their strategies performance, which can be easily performed using Python.

----

_Sources/Further readings:
Cin Sam (2021), Winning with Python — Getting Started with Quantitative Trading.
Nick Lioudis (2021), Moving Average Strategies for Forex Trading.
James Chen (2021), Exponential Moving Average (EMA).
Daniel P. Palomar (2020), Backtesting Portfolios._
