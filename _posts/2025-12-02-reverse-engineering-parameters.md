---
layout: post
title: Reverse engineering match odds from team totals
usemathjax: true
categories: [Football, Betting]
---

This post moves on from the failure in the previous post, where I tried to find an edge in the over/under markets using the correct score market under the assumption that bias between these two markets is inherently different, one directionally and one spread around different results. In this post I will try to tweak this idea a little where instead of using correct score I will use total team goals market to test whether it contains any useful signal to use in a main market.

The general idea has not changed, it is just that the source of data for the model is changed, moving from correct score to the team total goals. I think of the team total goals odds as marginal distributions for the teams. I am assuming again that team goals distribution are negative binomial. And I will use ChatGPT to estimate parameters $$\ \lambda_{home}, \lambda_{away}, k $$ (global k). Then, using these two independent marginal distributions I can construct a joint distribution and compare it to the cs market. The next tweak will be to compare it to the draw price from 1X2 market in order estimate a correlation parameter.

I hope this idea has better prospects because the team totals market is essentially higher up the hierarchy due to its lower vig (7% vs 15-20%). I guess that finance people will dismiss this idea on the spot because I am trying to find edge in a main market (over/under) using information from secondary, less liquid market. Unlike the previous post which was more top down, this post is more bottom-up approach as I have started from the bookmaker odds and maybe I will build a toy world later. I have picked the recent Champions League match between Atletico and Inter Milan. I started by asking ChatGPT to estimate the $$\ \lambda_{home}, \lambda_{away} , k $$ from the odds.

```
| Goals |   Atletico  |   Inter  |
|------:|------------:|---------:|
|   0   | 4.31        | 3.36     |
|   1   | 2.81        | 2.62     |
|   2   | 3.84        | 4.27     |
|   3   | 7.13        | 9.47     |
|   4   | 16.67       | 25.93    |
|   5+  | 36.02       | 67.94    |
```

and it estimated $$\ \lambda_{atl} = 1.55, \lambda_{inter} = 1.29, k = 50 $$ so basically a poisson, k was estimated for both separately and for both it was around 50, but I wanted to use global one so just set it to 50. Now I have marginal distributions of the teams and I can construct some odds myself. Given these parameters, the home win is 0.434, draw is 0.247 and away win is 0.319. Comparing home win from the model to the AH -0.5 (Atletico) which was priced at 2.29 (~0.4367), almost the same. Then I used the correct score odds to see how the model differ from them (assuming uniform vig). Odds in the table below, rows are Atletico goals and columns are Inter goals.

```
|   |   0  |   1  |   2  |   3  |  4   |
|---|-----:|-----:|-----:|-----:|-----:|
| 0 | 11.73| 11.53| 17.13| 35.33| 86.54|
| 1 |  9.60|  6.71| 11.69| 24.55| 62.11|
| 2 | 12.18|  9.83| 14.71| 30.29| 75.35|
| 3 | 21.77| 17.64| 25.81| 52.02|    NA|
| 4 | 47.69| 39.26| 55.99|    NA|    NA|
| 5 |    NA| 98.14|    NA|    NA|    NA|
```

After removing vig (again, assuming uniform vig) I get 0.42, 0.268 and 0.312 for Atletico win, draw or Inter win respectively. This is slightly lower than 0.434 suggested by the model, or by Atletico -0.5. But the biggest discrepancy is at the draw. I guess this was the first hint to correct for correlation. Using ChatGPT I created a heat map of the differences between model and implied probabilities from cs odds after removing vig.

![heatmap](/assets/atleti-inter.png)

It looks very symmetric. The 0-0 and 1-1 discrepancies are very big, roughly 1%. I am not sure if I should modify parameters to fit to these implied probabilities. Also, I have hard time figuring it out exactly because vig could be directionally different across markets, and we have three markets; Atletico total goals, Inter total goals and the correct score goals. One thing I am certain about is that the model should not be able to find any +EV in the CS market, which is indeed the case. 

I asked ChatGPT to compute implied probabilities of 1X2 from the CS market under two different scenarios; uniform - same overround on each score, draw - higher overround on 0-0, 1-1 and on a little extra on long tails, tails - high overround on high scoring goals.

```
|  vig  |   1   |   X   |   2   |
|-------|------:|------:|------:|
|uniform| 0.42  | 0.268 | 0.312 |
|draw   | 0.437 | 0.238 | 0.324 |
|tails  | 0.41  | 0.283 | 0.307 |
```

For the draw scenario, ChatGPT assumed vig is loaded mostly into the draw and some in the tails. I have only intuition here to trust, I believe that the draw scenario is very unlikely. Market odds for 1X2 is 2.28, 3.49, 3.25. Given these odds, under the draw scenario it would mean draw odds should be around 4.2 and Inter odds will be +ev which I find highly unlikely. There are many other scenarios possible for vig to be stacked but I get the feeling that model draw probability should be higher, and maybe a hint that Atletico is over priced.

Additional, the implied probability of draw by 1X2 market, without removing vig is $$\ 1/3.49 = 0.2865 $$ while the model probability for draw is $$\ 0.247 $$. That leaves me with $$\ 0.2865 - 0.247 = 0.0395 $$, it is larger than the vig for 1X2 market which stands at 3.2%. Again, I find it unlikely that vig will stacked completely at draw but still I should be cautious making any assumptions. I asked ChatGPT to estimate correlation parameter and fix the model draw probability to match market implied probability. It estimated $$\ \rho = 0.17 $$ and it fixed the model final probabilities to 0.4165, 0.2774, 0.3061. 

I have followed the odds until closing, and indeed the odds for Atletico drifted from 2.28 to 2.7 which is like 6.8%. That is huge. The draw odds stayed almost the same, 3.48. Inter odds also drifted from 3.25 to 2.70. If it is not just noise, then it looks like using a secondary market and a main market, could actually produce signal for another main market. But this was just one test case and it could have easily been noise, I will need much more data for that. In the next post I will elaborate why given my assumptions on previous posts, my tendency is to drop this pipeline altogether.
