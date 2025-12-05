---
layout: post
title: Trying to tweak the strategy
usemathjax: true
categories: [Football, Betting]
---

Moving on from the failure in the previous post, where I tried to find edge in the over/under markets using the correct score market under the assumption that bias between these two markets are inherently different, one directionally and one spread around different results. In this post I will try to tweak this idea a little where instead of using correct score I will use total team goals.

The general idea has not changed, it is just that the source of data for the model is changed, moving from correct score to the team total goals. I think of the team toal goals odds as marginal distributions for the teams. I will use chatgpt to estimate parameters $$\ \lambda_{home}, \lambda_{away}, k $$ (global k). Then, using these two marginal distributions I can construct a joint one and compare it to the cs market. The next tweak will be to compare it to the draw price from 1X2 market in order estimate a correlation parameter.

I hope this idea has better prospect because the team totals goals is slightly higher up the hierarchy than correct score market and the vig is much lower as well, 7% vs 15-20%. I guess that finance people will dismiss this idea on the spot because I am trying to find edge in a main market (over/under) using information from secondary, less liquidy market. This time time I have started immediately from the odds and may build a toy world later. I have picked the recent champions leauge match between atletico and inter milan. I started by asking chatgpt to estimate the $$\ \lambda_{home}, \lambda_{away} , k $$ from the odds.

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

and it estimted $$\ \lambda_{atl} = 1.55, \lambda_{inter} = 1.29, k = 50 $$ so basically a poisson, k was estimated for both separately and for both it was around 50, but I wanted to use global one so just set it to 50. Now I have marginal distributions of the teams and I can construct some odds myself. Given these parameters, the home win is 0.434, draw is 0.247 and away win is 0.319. Comparing home win from the model to the AH -0.5 (atletico) which was priced at 2.29 (~0.4367), almost the same. Then I used the correct score odds to see how the model differ from them (assuming uniform vig). Odds in the table below, rows are atletico goals and columns are inter goals.

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

After removing vig (again, assuming uniform vig) I get 0.42, 0.268 and 0.312 for atletico win, draw or inter win respectively. This is slightly lower than 0.434 suggested by the model, or by atletico -0.5. But the biggest discrepancy is at the draw. I guess this was the first hint to correct for correlation. Using chatgpt I created a heat map of the differences between model and implied probabilities from cs odds after removing vig.

![heatmap](/assets/atleti-inter.png)

It looks very symmetric. The 0-0 and 1-1 discrepancies are very big, roughly 1%. I am not sure if I should modify parameters to fit to these implied probabilities. Also, I have hard time figuring it out exactly because vig could be directionally different across markets, and we have three markets; atletico total goals, intel total goals and the correct score goals. One thing I am certain about is that the model should not be able to find any +EV in the CS market, which is indeed the case. 

I asked chatgpt to compute implied probabilities of 1X2 from the CS market under two different scenarios; vig packed in the 0-0, 1-1, vig packed in the tails. So we have

```
|  vig  |   1   |   X   |   2   |
|-------|------:|------:|------:|
|uniform| 0.42  | 0.268 | 0.312 |
|draw   | 0.437 | 0.238 | 0.324 |
|tails  | 0.41  | 0.283 | 0.307 |
```

For the draw secnario, gpt assumed vig is stacked mostly in the draw and some in the tails. I have only intuition here to trust, I believe that the draw scenario is very unlikely. Market odds for 1X2 is 2.28, 3.49, 3.25. Given these odds, under the draw scenario it would mean draw odds should be around 4.2 and inter odds will be +ev which I find highly unlikely. There are many other secnarios possible for vig to be stacked but I get the feeling that model draw probability should be higher, and maybe a hint that atletico is over priced.

Additional, the implied probability of draw by 1X2 market, without removing vig is $$\ 1/3.49 = 0.2865 $$ while the model probabilitiy for draw is $$\ 0.247 $$. That leaves me with $$\ 0.2865 - 0.247 = 0.0395 $$, it is larger than the vig for 1X2 market which stands at 3.2%. Again, I find it unlikely that vig will stacked completley at draw but still I should be cautious making any assumptions. I asked gpt to estimate correlation parameter and fix the model draw probability to match market implied probability. It estimated $$\ \rho = 0.17 $$ and it fixed the model final probabilities to 0.4165, 0.2774, 0.3061. 

I have followed the odds until closing, and indeed the odds for atletico drifted from 2.28 to 2.7 which is like 6.8%. That is huge. The draw odds stayed the almost the same, 3.48. Inter odds also drifted from 3.25 to 2.70. It could be that there is signal to be discovered in secondary markets and that can be useful for main markets. But this was just one test case and it could have easily been just noise that I am looking at. In the next post I will elaborate why given my previous assumption, this pipe line could not be profitable either.


