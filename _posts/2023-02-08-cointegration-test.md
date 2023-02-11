---
layout: post
title: Using cointegration to measure the co-movement of assets (Part 1)
usemathjax: true
categories: [Trading, Crypto, Cointegration]
---

I came across this [tweet](https://twitter.com/nik_algo/status/1620412692560490496) from some anon on twitter directing to this [paper](https://palomar.home.ece.ust.hk/MAFS5310_lectures/slides_pairs_trading.pdf) of Daniel P. Palomar from Hong Kong university. To reiterate from the paper, he suggests testing for cointegration instead of testing for assets returns correlation. While correlation characterizes short term co-movement, cointegration refers  to the long term co-movement. I think, if I understand correctly then the idea behind cointegration is to check for the difference between noramlized prices of assets and see if the difference is stationary. If it is then one can just regress the price of asset 1 over the price of asset 2 (with OLS) since they will converge at some point.

I'm not sure I fully understand it but I think both ways (correlation and cointegraiton) serves as way to measure co-movement of two assets but cointegration allow to see co-movement without the need to test it over different timeframes as with correlation.

To test for cointegration, you need to first normalize the price of both assets. A noramlized price is $$\ \tilde{p_t} = \frac{p_t}{p_0} $$ then, compute the difference $$\ \tilde{p_{1t}} - \tilde{p_{2t}} $$ of the noramlized prices of the assets. If the difference is stationary (mean reverting) then I may be able to model the two assets prices $$\ y_1, y_2 $$ as follows:

$$\ 
y_{1t} = \alpha \cdot x_{t} + \epsilon_{1t} \\
y_{2t} = x_{t} + \epsilon_{2t} \\
$$

With $$\ \epsilon_1, \epsilon_2 $$ are i.i.d. residual terms. And I use OLS to estimate the $$\ \alpha $$ which is the secrent ingridient.

I'm going to try and put it to the test. I used Binance API to download 5 minute candles, from Nov 1, 22' to Feb 1, 23', of every coin listed on binance. Had approx ~2200 coins. Unfortunately, mainy of the requests I sent to their server for the candle data, returned empty handed. So I filtered any of those and left with 1453 coins. I saved the data of each coin into its own CSV file so I will be able to access it later. 

I wanted to check every possible combination, which summed up to more than one million combinations. Unfortunately it was unnecessary as I forgot many assets are quoted in almost identical assets. For example, BNB-USDT, BNB-BUSD, BNB-USDC, BNB-DAI. USDT, BUSD, USDC & DAI are stable coins meant to follow the US dollar.
Anyway, I then ran a script checking the NPD for each two coins. $$\ NPD = \sum_{t=0}^T (\tilde{p_{1t}} - \tilde{p_{2t}})^2 $$ and saved the result for each possible combination into a file. So at first it doesn't look very promising. The highest ranked NPD coins pairs were actually currencies (AUD, EUR, USD). 

I have attached below some examples of pairs I have found. Some seems to be moving together while not necessarily having high correlation, although for these cases, it seems one of the coins are just not liquid enough and that what may cause it.

![normalized prices movement and difference](/assets/cointegration/DATAUSDT-OMUSDT.png)
Correlation of this pair is around $$\ 0.20 $$. Correlation plot:

![correlation plot](/assets/cointegration/DATAUSDT_OMUSDT_CORRELATION.PNG)

Others, seems to be highly correlated anyway, so I am not sure if cointegration has any additional value.
Correlation $$\ ~0.55 $$
![normalized prices and movement](/assets/cointegration/XLMUSDT-NEOUSDT.png)

I still have a very long list to go over so to be continued in the next note.
