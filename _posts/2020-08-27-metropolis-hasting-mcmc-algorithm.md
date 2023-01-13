---
layout: post
title: "Metropolis-Hasting sampling algorithm"
categories: Statistics Python MCMC 
usemathjax: true
---

MCMC sampling method allow to sample directly from the posterior without the need to know its shape or any other properties. Remember that the definition:

$$ P( \theta \ | \ data) = \frac{P(data \ | \ \theta) \times P(\theta)}{P(data)} \propto P(data \ | \ \theta) \times P(\theta) $$

What I'm actually doing is applying the last part only and directly its distribution. Remember that the denominator is just a normalizing constant (making sure the distribution sums up to 1) so we can ignore it.

The metropolis hasting method is quite simple. You start with an empty list of parameters.

1. At first, you pick a random point from the prior distribution.
2. Evlaluate:
$$likelihood \times prior = P( data \ | \ prior) \times P(prior) $$
3. Pick new random point from the prior and evluate again as in step 2.
4. Compute the ratio between the two: (new posterior)/(old posterior)
5. If ratio is bigger than 1 or it is bigger than a random number generated from uniform (in the example below) then you add it to your parameters list.
6. If ratio is smaller than one and smaller than random generated number then add your current number (again) to your list and repeat all the steps above.

In this example I'm going to estimate the probability of a coin landing heads given I have observed 7 heads and 3 tails.
I will set a beta prior with parameters alpha = 1, beta = 1 what makes it exactly the same as uniform distrubtion.

Beta distributions for reference: 

![beta distbution with different parameters](/assets/beta.jpg)



```python
import numpy as np
import scipy.stats as ss

def metropolis_hasting(N, a_beta, b_beta):
    
    parameters = np.zeros(N)
    start_p = 0.5

    for i in range(N):

        new_p = np.random.beta(a_beta, b_beta, 1)
        old_prior = ss.beta.pdf(start_p, a_beta, b_beta)
        new_prior = ss.beta.pdf(new_p, a_beta, b_beta)

        posterior_old = ss.binom.pmf(7, 10, start_p) * old_prior
        posterior_new = ss.binom.pmf(7, 10, new_p) * new_prior

        ratio = posterior_new / posterior_old

        if ratio >= 1 or ratio >= np.random.uniform(0, 1, 1):
            parameters[i] = new_p
            start_p = new_p
        else:
            parameters[i] = start_p
        
    return parameters

```


The good thing about choosing beta prior is that you can actually deduce the posterior analytically. I won't go into the details here but what's important to remember is that given a beta prior and a binomial likelihood, the posterior will also be beta with parameters
$$ \alpha_{posterior} = \alpha_{prior} + x \ \ \ , \ \ \  \beta_{posterior} = \beta_{prior} + n - x $$
Where x is the number of success and n is number of trials

Here is a comparsion of analytically deduced posterior and the MCMC results

![mcmc metropolis hasting result](/assets/mcmc.png)


