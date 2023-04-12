---
layout: post
title: Cointegration continued and deming regression
usemathjax: true
categories: [Trading, Crypto, Cointegration, Deming, Regression]
---

I came across this tweet of Ernest Chan where he advised to use deming regression instead of OLS to find the best regression line when measuring cointegration. OLS assumes no noise for the predictor and only for the predicted variable. So it finds the regression line that reduce the distances as measured vertically. Deming regression allows for noise of both the predictor and the predicted variable with extra parameter, $$\ \delta $$. $$\ \delta $$ is the ratio of the standard deviations of variables noise. if $\ y^*_i, x^*_i $ are the "true" values:

$$\ 

y_i = y^*_i + \epsilon_i \\
x_i = x^*_i + \eta_i 
\\ \\
\delta = \frac{\sigma^2_{\epsilon}}{\sigma^2_{\eta}}

$$

I wanted to put it to the test so generated some data and plotted it The dashed line is the true regression line. With the generated data I have $$\ \frac{\sigma^2_{\epsilon}}{\sigma^2_{\eta}} = \frac{1^2}{0.5^2} = 4 $$

```{r}

set.seed(1)
n <- 30
real_slope <- 2

x <- rnorm(n)
y <- x * real_slope + rnorm(n)
x_noise <- x + rnorm(n, sd = 0.5)

```

![plot](/assets/deming/deming_0.png)

For the deming regression function I have used both `MethComp` function and one I have written myself using wikipedia formulas. I have just decided to plug in random values and see the results:

| intercept  | real_slope | type               |
|------------|------------|--------------------|
| 0.01116014 |	2.082388  |	Deming, delta = 1  |		
| 0.01942136 |	2.022349  |	Deming, delta = 2  |		
| 0.02956800 |	1.948608  |	Deming, delta = 4  |	
| 0.05577043 |	1.758179  |	OLS                |

Clearly all of the deming models estimated the slope better than OLS model. Interesting to see that the model with delta 2 has better estimation of the real slope than the model with delta set to 4 which is the true value. This model is determinstic so I think it is something with how I generated the data, could be the only option. Also looking at the output of the both models:

```
> odr_model_2 # delta 2

        Intercept             Slope sigma.tbl$x_noise       sigma.tbl$y 
       0.01942136        2.02234920        0.38545088        0.54510986 

> odr_model_4 # delta 4

        Intercept             Slope sigma.tbl$x_noise       sigma.tbl$y 
         0.029568          1.948608          0.333395          0.666790 
```

I think the delta 2 model estimated the x noise better and delta 4 model estimated the y noise better? Not sure. 

![plot](/assets/deming/deming_1.png)!

Now to get the idea of difference between OLS and the deming regression I have plotted both models result and their distance. Since OLS doesn't allow for measurements in the predictor, the distances are measured vertically

![ols](/assets/deming/deming_2.png)

and since deming regression allows for predictor noise the distances can be measured diagonally.

![deming_2](/assets/deming/deming_3.png)

Conitinuing with the first part of cointegration, I spent the evening looking at graphs of coins pairs and found several that seemed like moving together yet not highly correlated. The best one I believed I have is ORNUSDT-TROYUSDT since both coins have high volumes.

![plot](/assets/cointegration/TROYUSDT-ORNUSDT.png)

with low correlation of 0.16.

![correlation](/assets/cointegration/TROYUSDT_ORNUSDT_CORRELATION.png)

Moving on as the paper suggests, I ran a simple regression (OLS) over the regular prices of these assets and here is the model summary:

`slope=302.953, intercept=0.0247, rvalue=0.90, pvalue=1.933645760589192e-188, stderr=6.329`

The slope is what I defined in part 1 as the $$\ \alpha $$ (or $$\ \gamma $$ in the paper) and the intercept is the $$\ \mu $$. Also computed:  $$\ R^2 = 1-\frac{SSE}{SST} = 0.8214 $$. Haven't checked any p-values and standard error for the intercept or the slope.

![regression](/assets/cointegration/TROYUSDT-ORNUSDT-regression.png) ![residuals plot](/assets/cointegration/TROYUSDT-ORNUSDT-residuals.png)

Looking at the residuals chart it seems homoscedasticity (constant error variance for $$\ \hat{y_i} $$ conditioned on $$\ x_i $$) assumption is violated. Although ADF test result for the residuals $$\ \hat{\epsilon_i} $$, the statistic is -3.406 with p-value of 0.0107.

I wanted to try deming regression to see the results. So I measured the rate returns of both coins. The ratios of the standard deviations of those returns turns out to be $$\ 0.992 $$ so basically 1. The result is plotted below:

![assets deming regressions](/assets/cointegration/cointegration_deming.png)

Using only my eyes I can not be convinced which regression fits better here? I guess should test it out on future observations. Also not clear to me the distribution of $$\ \hat{\ y} $$ . I guess the next step is computing the std of each observation given its expected value and if it's too far away from the expected value then maybe betting on a mean reversion.
