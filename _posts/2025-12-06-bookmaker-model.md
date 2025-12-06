---
layout: post
title: The bookmaker model - conclusions
usemathjax: true
categories: [Football, Betting, Bookmaker]
---

In my previous post, I explained how I tried to estimate each team's goal-scoring parameter using the team total goals odds. I assumed that goals for each team come from a negative binomial distribution with parameters $$\ \lambda, k $$ where k is a dispersion parameter. I then estimated a correlation parameter, $$\ \rho $$ which adjusts the probabilities for draws and 1-0, 0-1. With the help of ChatGPT and Gemini, I used this approach to follow several matches: first estimating parameters and then tracking how the odds moved. In this post, I will explain why this approach cannot be used to find an edge in the main markets of 1X2, AH and Over/Under.

Using the team total goals odds, I asked ChatGPT to estimate the parameters of the Bundesliga match between Mainz 05 and Borussia Mönchengladbach. After a day or so, I checked the same market again to see if and how it had moved. Below are the records of two different timestamps for both teams, the first one is from 2025-12-03 10:55 and the second one from 2025-12-04 22:37 

For Mainz 05


| Goals  | Earlier Odds | Later Odds | Change       |
| ------ | ------------ | ---------- | ------------ |
| **0**  | 4.310        | 4.510      | **↑ 0.200**  |
| **1**  | 2.840        | 2.890      | **↑ 0.050**  |
| **2**  | 3.790        | 3.740      | **↓ –0.050** |
| **3**  | 7.080        | 6.770      | **↓ –0.310** |
| **4**  | 16.700       | 15.520     | **↓ –1.180** |
| **5+** | 38.530       | 34.460     | **↓ –4.070** |


For Borussia Monchengladbach


| Goals  | Earlier Odds | Later Odds | Change       |
| ------ | ------------ | ---------- | ------------ |
| **0**  | 3.990        | 3.830      | **↓ –0.160** |
| **1**  | 2.760        | 2.730      | **↓ –0.030** |
| **2**  | 3.900        | 3.950      | **↑ 0.050**  |
| **3**  | 7.740        | 8.060      | **↑ 0.320**  |
| **4**  | 18.990       | 20.350     | **↑ 1.360**  |
| **5+** | 42.490       | 47.790     | **↑ 5.300**  |



For Mainz 05 we observe a decrease in the implied probability of 0,1 and increase in 2, 3, 4, 5+ goals. For Monchengladbach we see almost a mirror image of this shift. I’m fairly confident that what I’m seeing here isn’t the bookmaker manually fixing these lines, but something happening further upstream. Translating this movement back into the parameters, it means that Mainz’s parameter increased, as probability shifted from 0–1 goals to 2+ goals. For Monchengladbach, the parameter decreased, with probability shifting from 2+ goals towards 0–1 goals. Converting the odds movement to implied probability movement (no devig), the changes pretty much mirror each other. Table with difference of implied probabilities before and after:

|           |     0     |     1     |     2     |     3     |     4     |     5+    |
|----------:|----------:|----------:|----------:|----------:|----------:|----------:|
|Mainz      |  0.0102891|  0.0060919| -0.0035274| -0.0064675| -0.0045528| -0.0030654|   
|M. Gladbach| -0.0104700| -0.0039815|  0.0032457|  0.0051295|  0.0035192|  0.0026101|

I suspect the Asian Handicap (AH) market is the primary driver of this line movement. I can rule out the over/under market immediately because it should mostly impact $$\ \lambda_{total} $$ which if moved alone, should suppress the $$\ \lambda_{home}, \lambda_{away} $$ in the same direction. 1X2 is more difficult to rule out, but the draw odds stayed almost the same, moving only from 3.51 to 3.55 (based on a record I found on [goaloo](https://www.goaloo.com/football/german-bundesliga-fsv-mainz-05-vs-borussia-monchengladbach/1x2-odds-2799520))). This leaves me with one last market: Asian Handicap. I believe that money pouring in on Mainz forced the bookmaker to decrease the odds on Mainz and consequently increase the odds on Borussia Mönchengladbach, since the main AH lines are pure win/lose markets without a draw option. Unfortunately, I realised this too late to actually track the AH odds and confirm or reject this idea.

This led me to update my beliefs about how the bookmaker operates. After my previous posts, I ended up modelling a football match using a bivariate negative binomial distribution. In that model I estimate $$\ \lambda_{home} , \lambda_{away} $$, a shared $$\ k $$ and a correlation parameter $$\ \rho $$. I also argued in a previous post that bookmaker prices are internally coherent because they derive prices using the same model across markets. Then the bookmaker probably make some adjustments according to their expectation of market bias. What I think happens is that once the line opens, the bookmaker uses the main markets to move the parameters in this way:
1. Over/Under market - main line (usually 2.5/1.5) is used to update a $$\ \lambda_{total} , k $$ 
2. Asian Handicap - used to update $$\ \Delta_{\lambda} = \lambda_{home} - \lambda_{away} $$
3. 1X2 - once the others parameters are set, the draw is used to update the correlation parameter $$\ \rho $$

The bookmaker adjusts the parameters to accommodate these markets and, as a consequence, the secondary markets move as well. Since secondary markets have much higher vig, this probably protects the bookmaker from bettors finding any edge there. If everything I’ve written here is roughly correct, then I can drop my previous pipeline: there’s no realistic way to find a signal in a secondary market.

In reality, though, there are several big assumptions here that I hope are not all correct:
1. A single family of distributions is used across all markets
2. No extra shading or different vig structures/logic between markets
3. No desk-level tweaks on specific markets

If any of these assumptions does not, hold then there is still a gap to be exploited. I think what I should do next is estimate the parameters from main markets, create my own secondary market odds and compare them to actual odds. Then I'll analyse the residuals under different segmentations (by league, clear favorite games, etc.). If the residuals are always close to 0, then it will confirm that secondary market can not be used to find an edge. The main challenge now lies in acquiring the necessary data.

