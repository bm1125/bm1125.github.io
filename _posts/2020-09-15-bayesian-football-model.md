---
layout: post
title: Predicting premier league with bayesian multilevel model
categories: bayesian analysis football statistics hierarchical R
usemathjax: true
---

The twelve chapter of Statistical Rethinking by Richard Mclearth is about multilevel (or hierarchical) model. The multilevel models are sort of a compromise between independent and dependent clusters. I think one of the advantages of such model is that it allows me to say that I'm not really sure about the relationship between clusters. And it may be that one cluster has some infromative value on another cluster. In my case each cluster represent a team. I will try and make a multilevel model to predict english premier leauge games.

I believe it is easier to understand the differences between the models by looking at the mathematical notation.

(1)

$$ Goals_{k} \sim Poisson(\lambda) \\ log(\lambda) = attack_{i} + defense_{j} + \alpha_H * home \\
attack_i \sim Normal(0, 1) \\ defense_j \sim Normal(0, 1) \\ \alpha_H \sim Normal(0,1)
$$

Where $$ attack_i $$ is the attack of team *i* and $$ defense_j $$ is the defense of team *j* the $$ \alpha_H $$ is the intercept for home advantage.
If I ran this model, it will try to estimate those parameters for each team **independently**. A multilevel model is a better fit if you assume there is some dependency between the teams. Meaning, you can know more about team i, by knowing about team j.

The mathematical notation for a multilevel model:

$$ Goals_{k} \sim Poisson(\lambda) \\ 
log(\lambda) = attack_{i} + defense_{j} + \alpha_H * home \\
attack_i \sim Normal(\mu_H, \sigma_H) \\
defense_j \sim Normal(\mu_A, \sigma_A) \\ 
\alpha_H \sim Normal(0,1) \\
\mu_H, \mu_A \sim Normal(0, 1) \\
\sigma_H, \sigma_a \sim Exp(1)

$$

So basically it's like I set the attack and defense priors to be more adapative throughout the estimation process. Hence, it is like placing a prior on the priors. In this post I will do it with the rethinking package built by Richard Mclearth. Can also be done in pymc3 (python) and ofcourse rstan/pystan.

```r
	epl_model <- map2stan(
		alist(
			goals ~ dpois(lam)
			log(lam) <- a_team[team] + b_opponent[opponent] + ah * home
			a_team[team] ~ dnorm(mu_home, sigma_home)
			b_opponent[opponent] ~ dnorm(mu_away, sigma_away)
			ah ~ dnorm(0, 1)
			c(mu_home, mu_away) ~ dnorm(0, 1)
			c(sigma_home, sigma_away) ~ dexp(1)
		), data = final, iter = 4000, warmup = 500, chains = 2, cores = 2
```

Below is a plot with estimated parameters of teams attack and defense parameters. The x axis is for attack and y is for defense. Keep in mind, the lower the defense parameter , the better.
You can see the trend by looking at the plot but it is really hard (at least for me) to estimate the differences between the teams because it's log values.

![epl-bayesian-model-football](/assets/bayesian_epl.png)

I used data from 19/20 season, and since the season is already over and the league is going into matchday two, I'm going to try and predict the match between Wolves and Manchester City.
In order to predict a match I will sample the model parameters from the posterior and simulate a poisson process with those parameters. Remember $$ \lambda = \text{exp}(attack_i + defense_j + \alpha_H * home) $$

```r
	samples <- extract.samples(epl_model)

	goals_simulation <- function(team, opponent, home) 
			rpois(n = nrow(samples$a_team),
				lambda = exp(samples$a_team[,c(team)] + samples$b_opponent[,c(opponent)] + home * samples$ah)))

```

Wolves - Draw - Man City:

```
	[1] 0.2482303 0.2203352 0.5295773
```

I have also simulated the game using non-pooling (as in model (1))

```
	[1] 0.2208177 0.2269273 0.5512550
```

The regular model is actually closer to the implied probabilities on pinnacle (known as the benchmark for any model).

There are still a lot of problem I have to overcome before this model may have any use. Right now the multilevel model hasn't even converge properly. The effective samples number is too low, Not sure how to overcome this.