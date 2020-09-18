---
layout: post
title: Introduction to Bayesian Analysis
categories: 'Bayesian Analysis'
usemathjax: true
---

Although frequentist analysis is the what taught on most statistics courses I think it worth spending some time trying to understand the bayesian way. I'm not any expert, nor do my knowledge extensive enough to make any judgement. 

I think bayesian analysis is harder (at least for me) to understand than the frequentist. I've had a hard time understanding (still not sure I do) but I'm just going to explain what was the most appealing for me about bayesian analysis.

Think for example I would like to build a very simple model for predicting a football team goals. Suppose I have observed the following data:

```python
	array([3, 2, 2, 5, 2, 2, 3, 2, 1, 1, 4, 1, 0, 2, 3, 1, 1, 1, 5, 1])
```

where each data point represents the number of goals scored by my team in a specific match. Goals scored are discrete variable. Usually discrete variable are binomial, but since goals are binomial with a lot of trials and very few successes , I can use the poisson distribution instead.

Using maximum likelihood estimation (or mean, will give almost identical estimation) the estimated parameter λ will be 2.1 .




