---
layout: post
title: Trying use over dispersion for betting strategy
usemathjax: true
categories: [European Football, Soccer, Betting]
---

Previously I had worked on two different models trying to predict football 1X2 results. In the first model, the idea behind it was that maybe xG could provide extra signal and I found that it is easier to predict xG than goals. I had many iterations over, starting out with just a glm that also takes in xG prediction, then moved to predict directly goals difference using OLS instead of goals scoring, later on i picked multinomial logistic model when I understood that maybe SSE is not the right choice for loss function of a model and then due to model going crazy often, applied regularization (ridge regression). The other model was simpler, an ELO model where the actual score for each match was computed using 0,1,3 instead of 0,0.5,1 and for the expected score I used 1X2 closing line instead of the model predictions, $$\ 1/odds_{win} * 3 + 1/odds_{draw} * 1 $$. I made every mistake possible, worked a lot with gpt and gemini, discovered that both made up a lot of numbers when I asked them to run backtest. I think my most important take from this endeavor is that overall model accuracy matter less than finding true signal.

Upon listening to [@Gingfacekillah](https://x.com/Gingfacekillah) on a podcast saying (to paraphrase) *you build model to measure an effect and not in hope it will find one*. It resonated with me and I decided to drop my previous models even though P/L was positive. I knew I could not explain in simple terms why it is profitable and it's unlikely that my modeling abilities could beat the market, even if it was just MLS and J1 leagues. Maybe later I'll write a post about that attempt. Anyway, I think it's at that point I understood that I need to approach this betting challenge differently. Up to this point I treated betting as a pure prediction problem which I believe is a mistake. it’s a market problem first.

Because I’d just spent time thinking about over-dispersion, I got fixated on the over/under markets and assumed they were biased due to their popularity among recreational bettors. After a lot of messing around I managed to finalize an idea that I believe could work. I will use correct score and asian handicap odds to estimate parameters and look for discrepancy with over/under markets. My hope was that since bias in over/under markets are directional and in the CS the bias is spread around and less directional.

Technically, I was thinking of using the CS odds to estimate initial goal scoring parameter for each team under a poisson assumption. Then using AH odds, I can see if using these parameters, the CDF of $$\ \mu = \lambda_{HOME} - \lambda_{AWAY} $$ decays faster than suggested by the odds of some AH line k. Choosing some AH k for the home team $$\ k \in \{-0.5, -1.5, -2.5\} $$ and defining $$\ D = \text{Home Goals} - \text{Away Goals} $$, using normal approximation of skellam we can compute $$\ P(D \ge k) = 1- \Phi \left( \frac{k - \mu}{s \times \sqrt{\lambda_{HOME} + \lambda_{AWAY}}} \right) $$. Then s is a parameter of a skellam distribution (difference of two poisson processes). Converting AH odds to probability, the s can be extracted. If s > 1 then it suggests overdispersion. 

Working it out with chatgpt it has pointed out to a flaw in my reasoning by forcing a poisson model onto market prices that may very well not be derived from poisson, and then look for evidence it is not poisson using the AH odds. The pipeline suggested by chatgpt and that I eventually adopted, was to start by assuming over dispersion and to estimate $$\ \lambda_{HOME}, \lambda_{AWAY}, k $$ from the CS odds and drop the AH part or just use it as weak signal for extra verification. Then using these parameters I can estimate over/under odds and look for an edge. Again, hoping for bias in the over/under market. To test this idea gpt suggested we start by testing it in a sandbox.

I created a toy world where the true underlying goal scoring process is negative binomial with known $$\ \lambda_H, \lambda_A, k $$ and the bookmaker has one model which sets the odds for all of its markets. I will add 5%, 15% vig to the over/under and CS markets respectively, and add bias to the over/under market. To summarize: draw true parameters (randomly) -> create score distribution for those parameters (using negative binomial) -> convert them to true odds for each market -> add vig -> add bias to over/under market (optional). At first, I applied vig to CS market uniformly, which is not true but that will be adressed later.

The parameters of each match will be drawn as as follows: 
```{r}
  
  mu_total <- runif(1, 2.0, 3.6) # total goals around 2.5–3.2
  home_share <- rbeta(1, 3, 2)  # mean ~ 0.6 , bias home to have a bit more than away
  mu_home <- mu_total * home_share
  mu_away <- mu_total * (1 - home_share)
  k <- runif(1, 8, 24)  # dispersion parameter k: k -> infty, then it will be Poisson

```

Then, estimation of mu's and k will be done with r optim function using BFGS which if I understand correctly is some sort of gradient descent, but thats beyond my level of understanding. The function gpt wrote for me just tries to minimize $$\ \sum_{i,j} W_{i,j} \left( log p_{i,j}^{model} - log p_{i,j}^{market} \right)^2 $$ where W_ij is a weight that is set to be smaller (to 0.5 vs 1) for scores higher than 5.

Below is a small sample of outputs from the process (each column is an iteration). I think under these ideal conditions (uniform vig) the parameters estimation is quite good. k is the size parameter of negative binomial distribution. 

```{text}
|                           |       |       |       |      |       |
|:--------------------------|:------|:------|:------|:-----|:------|
|real_mu_home               |2.349  |2.093  |2.186  |1.592 |2.546  |
|real_mu_away               |1.151  |0.686  |0.766  |1.866 |0.584  |
|real_k                     |19.669 |10.314 |10.325 |8.965 |16.174 |
|predicted_mu_home_censored |2.339  |2.085  |2.176  |1.586 |2.535  |
|predicted_mu_away_censored |1.148  |0.685  |0.764  |1.858 |0.584  |
|predicted_k_censored       |20.971 |10.641 |10.668 |9.366 |16.872 |
```

The first challenge is to see that this pipeline does not produce fake signals, making sure that in a world with no shadowing of over/under markets at all, I will not find any bets. I model over/under bias by drawing $$\ \phi \sim N(0, \phi_sd) $$. If its positive the over side will be shadowed (shorter odds) and if negative, the under will be. Since its just simulation, I know the true odds and I can compute true EV for any bet I have "found" using the actual true parameters. This way I can label incorrect signals that I have found. After running the process 5000 times (5000 matches) with 0 no shading at all for the over/under markets, these are the results:

```{text}
n                  = 5000
n_flagged          = 33        # model says "bet over" on 0.66% of matches
fake_edge_count    = 33        # all of them are actually -EV
fake_edge_rate     = 1.0       # 100% of flagged bets are fake edge
avg_EV_hat         = +0.96%    # model thinks +1% on those
avg_EV_true        = -4.76%    # reality is -4.8% on those
```

I rerun this simulation but this time I actually pass on the true k to the bet EV estimation, to see if it still finds fake bets. It indeed eliminated all the fake bets under phi_sd = 0. Meaning the 33 fake signals found here are caused by wrong estimation of k parameter. I think when examining some results by eye, I did see cases where k estimated to be much larger than it truly is which means we assume the process to be poisson while it was not.

This was lab condition simulation so did not mean much. For it to be more realistic, I added some probability bumps to different scores in the CS odds, making it harder to estimate true parameters. Specifically, added 10-20% to the following score lines: 1-1, 2-1, 1-2, 2-2, 4-0, 0-5. I also modified estimation process to assume there are some bumps in specific scores, 1-1, 2-1, 2-2. I have also set phi_sd to 0.025, so this time there will be shadowing added randomly. I have also set threshold at 0 (meaning it will take any bet with estimated positive EV), the results were disastarous with ~75% fake signals. The correlation between estimated EV and true EV was high though, around 95%. So from now its mostly playing with threshold d and phi_sd. With phi_sd = 0.03 and threshold 1% it does okay, results below

```{text}
|    n| n_flagged| fake_edge_count| fake_edge_rate| avg_EV_true_on_flagged| avg_EV_hat_on_flagged|       cor|
|----:|---------:|---------------:|--------------:|----------------------:|---------------------:|---------:|
| 5000|       437|             172|      0.3935927|              0.0055328|             0.0280574| 0.9583606|
```

So now I was eager to see how it will do in the real world. Fortunately, the market showed me the door immediately. It died at first contact with the market. The immediate clue was when the model found 10.5% edge on one of the over bets, I think the instinct of any sane person is to immediately understand something wrong with the model. Then I checked some more lines and different matches. For all of them the model was finding +EV bets on the over picks. Given the magnitude, and the direction, these are clear signals that the model is way off. I was underestimating how much bookmakers shade long tails on CS market which led to inflation of the estimated parameters. Soul crushing.
