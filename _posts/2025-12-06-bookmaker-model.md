---
layout: post
title: The bookmaker model - conclusions
usemathjax: true
categories: [Football, Betting, Bookmaker]
---

In my previous post I explained how I tried to estimate each team goal scoring parameter using the team total goals odds. I assumed each team goals come from a negative binomial distribution with some $$\ \lambda, k $$ where k is a dispersion parameter. I then estimated a correlation parameter, $$\ \rho $$ which adjust the probabilities for draws and 1-0, 0-1. With the help of ChatGPT / Gemini, I have used this approach to follow several matches, by estimating parameters first and then tracking the odds movement. In this post I will explain why this approach can not be used to find edge in the main markets of 1X2, AH, Over/Under.

Using the team total goals odds, I asked ChatGPT to estimate the parameters of the Bundesliga match between Mainz 05 and Borussia Monchengladbach. And then, after a day or so (I think), I checked the team total goals odds again to see if and how they moved. Below is screenshot with two different timestamps, the first one is from 2025-12-03 10:55 and the second one from 2025-12-04 22:37 

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



For Mainz 05 we observe decrease in implied probability of 0,1 and increase in 2,3,4,5+. Monchengladbach is directly(!) the opposite. I have high conviction that what I observed here isn't bookmaker fixing these lines but something more upstream. If I translate this movement back to my parameters, this basically means that parameter of Mainz increased, as probability shifted to the 2+ goals from 0,1 and for Monchengladbach, the parameter has decreased as probability shifted to 0,1 goals. Converting the odds movement to implied probability movement, it looks like change pretty much mirror each other. Table with difference of implied probabilities before and after:

|           |     0     |     1     |     2     |     3     |     4     |     5+    |
|----------:|----------:|----------:|----------:|----------:|----------:|----------:|
|Mainz      |  0.0102891|  0.0060919| -0.0035274| -0.0064675| -0.0045528| -0.0030654|   
|M. Gladbach| -0.0104700| -0.0039815|  0.0032457|  0.0051295|  0.0035192|  0.0026101|

I think the immediate suspect here is the AH market that moved the line. I can rule out the over/under market immediately because it should mostly impact $$\ \lambda_{total} $$ which if moved alone, should supress the $$\ \lambda_{home}, \lambda_{away} $$ in the same direction. The 1X2 is more difficult to rule out but I observed that draw odds stayed almost the same, from 3.51 to 3.55 (I found a record for it on [goaloo](https://www.goaloo.com/football/german-bundesliga-fsv-mainz-05-vs-borussia-monchengladbach/1x2-odds-2799520)) This leave me with one last market, asian handicap. I believe that money pouring in on Mainz forced the bookmaker to decrease odds for Mainz and consequently increase odds for Borussia Monchengladback as the main lines of AH are no win/lose without draw option. Unfortunately, I figured it out too late to be able to actually track the AH odds and be able to confirm it or reject.

This led me to update my beliefs about the bookmaker operations. After my previous posts, I have eventually came to model a football match using a bivariate negative binomial distribution. I have estimated a $$\ \lambda_{home} , \lambda_{away} $$, a shared $$\ k $$ and some correlation parameter $$\ \rho $$. I also stated in a previous post that I believe bookmaker prices are internally coherent because they derive prices using the same model across markets. Then, the bookmaker will probably make some adjustments according to their expectation of market bias. What I think is happening is that once line opens, the bookmaker is using the main markets to move parameters in this way
1. Over/Under market - main line (usually 2.5/1.5) is used to update a $$\ \lambda_{total} , k $$ 
2. Asian Handicap - is used to update $$\ \Delta_{\lambda} = \lambda_{home} - \lambda_{away} $$
3. 1X2 - Once the others parameters are set, the draw is used to update the correlation parameter $$\ \rho $$

Bookmaker adjust parameters to accomodate these markets and consequently, the secondary markets are moving as well. Since secondary markets have much higher vig, its probably what protecting the bookmaker from bettors finding any edge in these markets. If all of what I have written here is true, then I can drop my previous pipeline as well as there is no way I can find signal in a secondary market.

In reality though, there are several big assumptions I make here that I hope are not all correct:
1. Single family of distribution used across all markets 
2. No extra shading, different vig structures/logic
3. No desk level tweaks on some markets

If any of these assumptions does not hold then there is still a gap to be exploited. I think what I should do next is estimate the parameters from main markets, create my own secondary market odds and compare them to actual odds. Then analysing the residuals under different segmentation (by league, clear favorite games, etc...). If residuals are always 0 then it will confirm that secondary market can not be used. The big challenge now is getting the data.

