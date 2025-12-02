---
layout: post
title: Over dispersion in football
usemathjax: true
categories: [Football, Betting]
---

I had saw a post of [@Gingfacekillah](https://x.com/Gingfacekillah) on X where he suggested to look at the over/under markets just like a stock option with a strike price (the line). I was not sure how or why but with the help of ChatGPT I decided to look into it, specifically in european football market.

To quickly recap, goals scored in a match originally modeled as two independent poisson process, where the model estimates attack and defense parameter for each team within a league, along with a home parameter which estimates the home advantage for the league. The end result is that the goals scored in some match will be $$\ \lambda_T = \lambda_H + \lambda_A $$ where $$\ \lambda_H = attack_H + defense_H + \text{home_adv} $$ and $$\lambda_A = attack_A + defense_H $$. Later on, additional modifications to this model were suggested by dixon and coles where they estimated an additional parameter, $$\ \rho $$ which shifted the probabilities among the 0-0, 0-1, 1-0, 1-1 scores due to some dependency.

Back to the over/under markets, the idea here is that the line quoted (i'll call it $$\ k $$) is a strike price, while $$\ \lambda_T $$ is the stock price. Then I can define any market where $$\ k \ge \lambda_T $$ as in the money and out of the money otherwise. The main property of a poisson distribution is that $$\ E[X] = Var(X) $$ and it gets interesting if this assumption is violated regarding goal scoring of some team. Assuming that we have a small over dispersion of goals, that is to say $$\ E[X] < Var(x) $$ that will make the probabilities of tail events higher. More practically, if $$\ k > \lambda_T $$, that means the $$\ P(\text{goals scored} > k) $$ will be slightly higher than suggested by poisson CMF, and if the $$\ k < \lambda_T $$ that means $$\ P(\text{goals scored} > k) $$ will be slightly lower than suggested by a poisson CMF. Below is a graph to illustrate it, the x axis is $$\ \lambda_T $$ and y axis is the probabilitiy of more than 2 goals in a match ($$\ k = 2 $$), under three different scenarios, two of which I inflated the variance a little but I believe it to be a reasonable inflation.

![convexity](/assets/convexity.png)

My initial attempt was to take some dataset I have of previous season of the MLS and see if betting over/under 2.5 line when data suggests over dispersion produces profit. That is, when both teams variance up to that specific match, were larger than the overall league variance up to that match, and estimated lambda bigger than line, I bet under, and when estimated lambda smaller than line, bet over. Surprisingly, it did actually produce profit, and a lot. Then when I see a breakdown by selection, only the over bets were profitable, and that where majority of the bets were as well. so I decided to test it against naively betting on the over in any match of the MLS in that season and indeed, profit was even higher. So I think thats when I understood that I need to take a step back. Before trying to estimate over dispersion using made up rules, I need to know if it's even possible to identify it, and beyond what level, I can no longer safely distinguish between a regular poisson and a slightly dispersed one. 

For this I will randomly generate 20 $$\ \lambda_i \sim N(1.8, 0.5^2) , i = (1,...,20) $$ where each $$\ \lambda_i $$ aim to serve as a team goal scoring parameter. Then for each $$\ \lambda_i $$ I will generate a two samples of size 38 each, one from poisson distribution and one from negative binomial distribution where I will slightly inflate the variance by 20%. Then I will see if I can eyeball over dispersion on a team level, and at what point (if at all) I can start see it. If I can not see it, I will inflate variance further until I can. While formal tests for over-dispersion exist, I prioritized visual intuition for this exploratory phase. Should also check it on a league level, to see if and when it is possible to recognize over dispersion. I think I also made here a big assumption that goals scored by team are independent which I as I stated in previously, it has already established by dixon and coles that it is not. But, this is a good way to know, with optimal conditions, how hard it is to identify over dispersion.

I generated some team parameters using a normal distribution with parameters below and checked that they do overall make sense:
```{r}
> set.seed(0)
> lambdas = rnorm(n = 20, mean = 1.8, sd = 0.5)
> lambdas
 [1] 2.431477 1.636883 2.464900 2.436215 2.007321 1.030025 1.335716 1.652640 1.797116 3.002327 2.181797 1.400495 1.226171
[14] 1.655269 1.650392 1.594245 1.926112 1.354039 2.017842 1.181231

```

I then sampled n = 38 from 20 different poission distribution, and also from 20 different negative binomial distribution while inflating the variance by 1.2. The `generate_nbinom_like` and `generate_poisson_like` functions output a matrix of size 20x38, each column is a team and each row is a match.

```{r}
compute_size <- function(lambda, inflator) { (lambda^2) / (lambda*inflator - lambda) }

generate_nbinom_like <- function(lambdas, var_inflator) {
  sizes = compute_size(lambdas, var_inflator)
  mapply(function(mu, size) rnbinom(n = 38, mu = mu, size = size), lambdas, sizes)
}

nbinom_like = generate_nbinom_like(lambdas, 1.2)
poisson_like = generate_poisson_like(lambdas)

# some sanity check
> apply(poisson_like,2,mean)
 [1] 2.184211 1.842105 2.789474 2.631579 1.842105 1.105263 1.131579 1.815789 1.421053 2.894737 2.184211 1.157895 1.210526
[14] 2.000000 1.315789 1.657895 1.973684 1.473684 1.789474 1.210526

> apply(nbinom_like, 2, mean)
 [1] 2.657895 1.131579 2.394737 2.526316 1.657895 1.000000 1.184211 2.026316 1.684211 3.605263 1.868421 1.526316 1.526316
[14] 1.236842 1.394737 1.684211 1.763158 1.342105 2.289474 1.210526
```

Below is a plot of the difference between running variance and mean (var - mean). I wanted to see how fast (if it all) value converge to 0 or something close to it. I have plotted both poisson and negative binomial processes side by side. Each line is a team and the y axis is match number (again 38 matches through the season). The dotted lines at 1, -0.5 are my attempt to eyeball an upper and lower bound for the difference. Again, maybe I could run some statistical test here or something complicated but am not interested in that. Basically it is really hard to tell the two? Also wonder what

![difference mean and variance plot](/assets/diff_plot.png)

I thought that given the team graph there is no reason for me to see any indication for over dispersion in an overall league graph. But to my surprise, it quickly reveals itself. The dotted lines are the true variance value for each distribution (4.83 for negative binomial and 4.12 for poisson). Which, to my surprise, even in the poisson case the variance does not equal the expectancy value. Working it out with gemini, the reason is that this is a mixture of poissons and jumping in between these poisson distributions induce some extra variance which then leads to higher variance. In this specific sample of lambdas, their variance is 0.2608. 
Anyway, it does look like for the poisson process, the variance does hover around the mean most of the league, while in the negative binomial case it quickly reveals itself to be much higher. 

![leauge](/assets/overall_league.png)

I think it is really not feasible to estimate dispersion on team level data. Even with 38 matches I could not see it and obviously I will not have 38 matches before start betting but more like 10-11. I worked it out with Gemini and it prodcued r code for me that will simulate a league with 20 teams and goals inflated by 1.2 as before. Then it tries to estimate var inflation by team (by just dividing variance of goals of each team by its mean) or just globally using GLM model (goals ~ team).
Gemini suggested to only try estimate over dispersion on a league basis and not by team. It also suggested me to not think of it as some team trait but as a system trait, because it is just an extra parameter to induce some uncertainty down to the predictions later. Results of the simulation below.

![gemini-simulation](/assets/per_team_vs_global.png)

