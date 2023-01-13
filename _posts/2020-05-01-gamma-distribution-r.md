---
layout: post
title: Gamma distribution in R
categories: R gamma probability
usemathjax: true
---

Since I get confused a lot when trying to use the gamma function in R and I tend to forget its properties so I wrote this simple reminder. The Gamma distribution is used to predict what is the probability of *k-th* events to occur in a specific time interval. It's important to remember, as you will mostly use only the cdf fucntion (As with most continiuous functions), so the question it is answering is, given some rate of events and some events count, what is the probability of observing this amount of events in chosen **T** time?

For example, Suppose I want to model number of goals a team will score in the next time interval. The R code is

```r
	pgamma(x, k, theta) //cdf 
```

Where **x** is the number of minutes, **k** is the number of goals and $$\theta$$ is the rate parameter. In this example lets say the team I'm following scores on average 1.95 goals per game and each game is 95 minutes (including additional time) then the $$\theta $$ parameter is $$ \frac{1.95}{95} $$

In some cases (in python I think) the parameter should be calculated the opposite way ie $$ \frac{95}{1.95} \approx 48.71 $$ minutes per goal

This comes in handy in live betting. Lets say for example that this team currently play against some other team and the game is in its 10th minute. I want to know what is the probability of seeing 2 more goals by the end of the game. Since we're at the 10th minute, I know there is 85 minutes left (with estimated additional time of 5 minutes). 

The following plot is the density plot although, at least for me, it is not really helpful as I find it hard to deduce anything out of it.

![gamma-density](/assets/gamma_plot.png)

plotting the cdf will be much more useful in order to see the actual probabilities.

![gamma-cumulative-density](/assets/gamma_cdf.png)

using R
```r
	pgamma(85, 2, 1.95/95)
```

```
	0.5205193
```