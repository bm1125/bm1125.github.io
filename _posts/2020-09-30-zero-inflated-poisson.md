---
layout: post
title: Zero inflated poisson regression
usemathjax: true
categories: pymc poisson regression bayesian
---

I think the best way to explain the idea behind the zero inflated poisson is with an example. As the name suggests, it is a model in which the observed data suggests that an observations with zero counts is more likeliy than suggested by a regular poisson model.
Lets say I want to model the amount of a goals a football player scores in a match. For the sake of simplicity I'm going to assume that the player is either playing all of the match or isn't playing at all. The data will probably suggests higher count of zero occurences as sometimes the player will not be playing due to various reasons. So what I need to model is two different processes. One is the probability of a player playing in some match $\ k $, the other is the rate (poisson process) of events.

In mathematical notation I'll say that the probability $$ x $$ goals in a match is :

$$ P(\text{Goals} = 0) = P(\text{Goals} = 0 \ | \ \text{Has Played}) \times P(\text{Has Played}) + P(\text{Goals} = 0 \ | \ \text{Not Played}) \times P(\text{Not Played})  $$

And if $$ x \ne 0 \\
\\
\\
P(\text{Goals} = k) = P(\text{Goals} = k \ | k \sim Pois(\lambda))
$$ 

So now I am going to generate some fake data. Lets suppose that I want to model goals by some player in every match when the player has 60% chance of playing in any particular game and his score rate for each game is 1. I'm going to generate data for 40 matches. 

```python
	data = np.random.binomial(1, 0.6, 40) * np.random.poisson(1, 40)
```

So it is pretty simple, the first part of the code will be $$ 0, 1 $$ vector where $$ 0 $$ indicate the player didn't play and $$ 1 $$ that he did.  The second part just simulate a poisson process. 

![zero-inflated-vs-poisson](/assets/zip_simulation.png)

Building the model using pymc is quite simple. In this case I have only two parameters to estimate. The $$ \lambda $$ of the poisson process and the $$\ p $$ , the probability of the player playing in a match.

```python

	with pm.Model() as zip_model:
    	# Priors - remember they are logit and log functions!
	    a = pm.Normal('a', 0, 1) 
	    lam = pm.Normal('lam', 0 ,1)
	    
	    lambda_ = pm.math.exp(lam) # Process rate of events
	    p = pm.math.invlogit(a) # Probability of playing in a match
	    
	    y = pm.ZeroInflatedPoisson('y', psi = p, theta = lambda_, observed = data['output'])
	    
	    trace = pm.sample(draws = 3000, tune = 1000, chains = 2, cores = 2)
```

Now lets see what is the mean of the estimations of each parameter.

```python
	logistic(trace['a']).mean() # Probability of playing in a match

	0.6047753681606323

	np.exp(trace['lam']).mean()

	0.7717652918488087
```

So the rate process is quite off to be honest (Remember I set it to 1). But then again it is only 40 data points , my guess is that it is really hard to estimate. Here is a plot of the posterior distribution for $$ \lambda $$

![posterior-lambda](/assets/lambda_est.png)

and finall, here's a comparsion of the observed data (the fake data I generated earlier) with a zero inflated model and a regular poisson model (Just took the average and computed probabilities).

![final-zip-results](/assets/zip_result.png)


|Observed|Zero-Inflated|Poisson|
|--------|-------------|-------|
|0.675|0.684500|0.637628|
|0.225|0.208833|0.286933|
|0.075|0.080333|0.064560|
|0.025|0.021000|0.009684|

For every possible value, the zero inflated model estimated probability much better than just using poisson average. Although both of them overestimated the probability of 1.