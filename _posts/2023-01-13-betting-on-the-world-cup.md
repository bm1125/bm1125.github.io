---
layout: post
title: Betting the world cup
categories: [world cup, betting, football]
usemathjax: true
---

I have always been fascinated with sports betting and professional bettors. I have tried several times in the past to do it professionally, either in quantitative or qualitative ways. I guess it is much harder nowadays to find an edge with quantitative models versus qualitative assessment. The literature I read stated that the market is very efficient for popular sports leagues. In the book soccermatics, [David Sumpter](https://twitter.com/soccermatics/) did find that draw odds in matches of equivalent teams were underpriced. That edge was quickly exploited by others and bettors, and it faded. There was one guest in the podcast [bet the process](https://podcasts.apple.com/us/podcast/bet-the-process/id1291010585) that claimed that are much more money to be made in qualitative ways.

In the past, I had some success betting in-play on over/under goals per match using gamma distribution with beta parameter as the number of expected goals output of a simple GLM. I had around ~115 bets with a return of 15%. It was tiring because I didn't have any tools at my disposal to automate the process, so eventually, I dropped it. During this world cup, I wanted to see if I had a sharp eye for market inefficiencies, so I scanned the market every day, looking for what seemed to be opportunities. I didn't have any system or model in place. I was reading pre-match analysis, used prior knowledge about some teams, and tried to predict the psych of the team for a particular match and if it could affect their game plan. I also had an assumption that national team games are harder to model.


|match                 |bet                      |odds |stake|outcome|
|:-------------------- |:---------------------   |----:|----:|------:|
|France vs Australia   |BTTS - Yes	             |2.44 |49.2 |TRUE   |
|Poland vs KSA         |1st Half - Draw	         |2.00 |50.0 |FALSE  |
|France vs Denmark     |1st Half - Under 0.5     |2.60 |20.0 |TRUE   |
|Argentina vs Mexico   |AH Home Team -1.5        |2.52 |40.0 |TRUE   |
|Brazil vs Switzerland |1st Half - Under 0.5     |2.94 |40.0 |TRUE   |
|Wales vs England      |Total Goals - Over 2.5   |2.04 |45.0 |TRUE   |
|KSA vs Mexico         |1st Half - Under 0.5     |3.15 |35.0 |TRUE   |
|Canada vs Mexico      |AH Home Team +0	         |3.25 |45.0 |FALSE  |
|Serbia vs Switzerland |1st Half - Under 0.5     |2.80 |45.0 |FALSE  |
|Argentina vs Australia|BTTS - Yes	             |2.82 |100.0|TRUE   |
|France vs Poland      |AH Home Team -1.5        |2.04 |30.0 |TRUE   |
|Morocco vs Spain      |Total Goals - Under 0.5  |8.60 |10.0 |TRUE   |
|France vs Morocco     |Total Goals - Over 2.5   |2.24 |50.0 |FALSE  |
|Argentina vs France   |Argentina to lift the cup|1.82 |537.0|TRUE   |


Overall it was a profitable world cup for me. Overall I had ten successful bets out of 14. It is convenient to confuse luck with skills, and I want to believe that I could be a sharp bettor, so I needed to test it. Assuming no skill at all, how lucky was I? I think the easiest way to answer this question is using a binomial model. The average odds for the bets listed above is 2.94, which mean that assuming bets are independent (which I guess can be questioned?), My success probability for every bet (trial) is 34%. Given a binomial r.v. with `n = 14, p = 0.34`, what is the probability for 10 or more successes in such a trial? Using R:

```r

sum(dbinom(c(10:14), 14, prob = 0.34))

>>[1] 0.004757905

```

Less than 0.5%! The Morocco - Spain bet immediately sticks out because it was a longshot bet with high odds compared to the rest. Doing the same test without this bet gives an average odds of 2.51. So the probability for 9 or more successes out of 13 trials with `p =0.4` for each trial is 3.2%.
Guess it was just luck after all :(



