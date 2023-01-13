---
layout: post
title: Using zero-inflated poisson to predict Messi's goals per game
categories: [messi, football, laliga, poisson, regression, bayesian, pymc]
usemathjax: true
---

My previous post was about a zero inflated poisson process where an observation of zero counts may be possible due to two different processes. After understanding (hopefully) the idea behind ZIP (zero-inflated poisson) models I wanted to try and see if I could use it to predict the number of goals a specific player is going to score in a specific match.

First, I had to collect the data. Since this type of data is not available at football-data.co.uk and api's services will cost me money I had to build a script that scrape the data from transfermarkt.com. The [script](https://github.com/bm1125/football-analysis/blob/master/transfermrkt.py) is available on my github. I didn't waste to much time polishing it so it is a bit complex.

A quick look at the data does reveal that the count seem to be a bit zero inflated because for he may not score a goal due to also not playing at all (or playing partly). The black box on top of the zero column is the number of matches where he didn't play. 

![messi-observed-goals](/assets/messi-zip/messi.png)

Here's a quick view for the data, where 0 in the position mean that he has not played (bench), -1 means he wasn't even in the squad.

| date                | position   |   goals |   minsplayed |
|:--------------------|:-----------|--------:|-------------:|
| 2004-10-16 00:00:00 | RW         |       0 |            8 |
| 2004-10-24 00:00:00 | LW         |       0 |           19 |
| 2004-10-30 00:00:00 | 0          |       0 |            0 |
| 2004-11-06 00:00:00 | 0          |       0 |            0 |
| 2004-11-14 00:00:00 | 0          |       0 |            0 |
| ..... |

Now to recall the mathematical definition of the model is:

$$ \text{Messi Goals in Match k} \sim ZIPoisson(\rho, \lambda) \\
logit(\rho) = \alpha_{\rho} \\
log(\lambda) = \alpha_{\lambda} \\
\alpha_{\rho} \sim Normal(0, 1) \\
\alpha_{\lambda} \sim Normal(0, 1)
$$

The problem with this dataset is the observations counted over different time intervals so I also had to the estimate the amount of minutes he will play in a game. So it is one more parameter to estimate.

```python
with pm.Model() as zip_model:
    
    a_mu = pm.Normal('a_mu', 0, 1)
    a_sigma = pm.Exponential('a_sigma', 1)
    
    ap = pm.Normal('ap', a_mu, a_sigma)
    p = pm.math.invlogit(ap)
    
    y = pm.Binomial('y', n = 90, p = p, observed = final.minsplayed)
    
    al = pm.Normal('al', 0 , 1)
    lambda_ = pm.math.exp(al)
    
    goals = pm.ZeroInflatedPoisson('goals', psi = pm.math.invlogit(y), theta = lambda_, observed = final.goals)
    
    trace = pm.sample(draws = 4000, tune = 1000)
```

Eventually, the posterior distribution of our $$ \lambda $$ parameter (goals per one game):

![messi-lambda](/assets/messi-zip/messi-lam.png)

and the posterior for the probability of Mesii playing in a specific game:

![messi-rho](/assets/messi-zip/messi-p.png)

When I retropredict (trying to predict the observations we had with the model):

![messi-model-results](/assets/messi-zip/messiresults.png)

I'm disappointed. It misses just as much as a regular poisson (predicting goals using average per game)
Maybe a combination of both model could balance out the underestimation and overestimation of those models.


