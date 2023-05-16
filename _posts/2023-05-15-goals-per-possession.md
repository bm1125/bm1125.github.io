---
layout: post
title: Goals per possession
usemathjax: true
categories: [Football, EPL, Laliga, Possession, Betting]
---

For a long time, I had a thesis I wanted to test. My thesis was that some teams can convert little possession into goals much better, hence those teams will be underestimated against some better opponents.

The plan is to create a new feature, I'll call it goals per possession which is calculated for each match by taking the goals a team scored in a match and dividing it by the possession percentage. I think of the result as the number of goals scored for each percentage of possession of the ball. The is a little problem though, since there is more than one way to measure possession in a football game. I used the data from [fbref](https://fbref.com) which stated that the possession is calculated as the percentage of passes attempted. To the best of my understanding, it is just successful passes divided by the total passes a team had. The odds data were taken from [football-data.co.uk](http://football-data.co.uk). The data sets of EPL & Laliga are available [here](https://github.com/bm1125/bm1125.github.io/tree/main/assets/possession-post/data).

At first, I was tempted to use xG (expected goals) since fbref has this stat for each match, provided by Opta. Eventually, I decided to use goals instead because I assume teams are not equal in terms of chance utilization. xG is just an average of all teams so a feature based on xG will produce more noise because I'd like to characterize each team independently. To test my assumptions, I have plotted below the mean of goals per game and expected goals per game for each team, ordered by the team position in the league (data is for season 2022-2023, up to matchday 3). Maybe the trend is not as clear as I expected yet it seems most of the top half teams do perform much better with a positive GF - xGF difference and the bottom half with a negative difference.

![xG - GF Difference](/assets/possession-post/01.png)

As stated above, my goal was to test whether high-converting teams (high goals per percentage possession) are more likely to win than low-converting teams when in both cases they are the underdogs. The average goal per percentage possession across all league matches is 0.02957 (n = 666). I decided to take teams with higher or equal to the league average and left out with following eight teams:

```
[1] "Arsenal"                  "Brentford"                "Brighton-and-Hove-Albion" "Liverpool"                "Manchester-City"          "Manchester-United"       
[7] "Newcastle-United"         "Tottenham-Hotspur"  
```

All of the teams cited above are already in the top half of the table so when I'll filter for the matches where the selected team is the underdog, it will most likely be only against other teams in this group.
I took a glance at the summarized data of the two groups (high converting and low converting, both underdogs):

|high_converting|wins|total|win_ratio|avg_odds|
|---------------|----|-----|---------|--------|
|FALSE          |53  |268  |0.1977612|5.945597|
|TRUE           |19  |65   |0.2923077|4.666923|


And their odds distribution for an underdog win

![underdog win](/assets/possession-post/02.png)

I think it is surprising for me as I expected the low-converting teams to have much higher odds (low implied probability to win) given they are from the top half of the table, so hopefully the market isn't pricing it in. Though it is only 65 observations in the high-converting group. Let's see the P/L for this strategy, again, betting on the underdog if the underdog is a high-converting team:

![P/L](/assets/possession-post/03.png)

This is alarming, to say the least. Not seems like steady growth but a few big jumps. The biggest one is somewhere around bet 31. Where Brentford won against Man City in an away match with odds 20.24. Also, this result is with look ahead bias since I'm using a parameter (goals per possession) measured at a time `t` and testing the strategy from `i = 1, ..., t`. It doesn't look very promising. I have uploaded a CSV file with selected bets available [here](https://github.com/bm1125/bm1125.github.io/blob/main/assets/possession-post/data/epl_selected_bets.csv) and summarized data:

|count|success|avg_implied_prob|
|-----|-------|----------------|
|65   |	19	  |0.2562889	   |


I guess this doesn't pass the statistic test at a 5/10/15% significance level. I test this strategy in Laliga. Why? I don't know. I don't have an actual good reason, I guess I should have used more EPL seasons instead since this hypothesis came to my mind after watching EPL games. Before I can move on, and avoid lookahead bias but still be able to use a stable estimation of the conversion rate from possession. I wanted to see approximately from which match day the parameter (conversion rate from possession) has stabilized. I calculated the moving average of the conversion rate for each team across the season:

![Conversion parameter moving average](/assets/possession-post/04.png)

I believe it is safe to assume matchday 15 is the point from which the parameter pretty much stabilized. Onto Laliga, I split the data on matchday 15. Meaning, computed the goals per possession for each team using their first 15 matches then selected all the teams that have above-average conversion rates (0.02537194)

```
 [1] "Almeria"         "Athletic-Club"   "Atletico-Madrid" "Barcelona"       "Espanyol"        "Girona"          "Rayo-Vallecano"  "Real-Madrid"     "Real-Sociedad"  
[10] "Valencia" 
```

![laliga-test](/assets/possession-post/05.png)

This is clear as night and day. Nothing to look further into. It is tempting to try and set different parameters for filtering the bets until I find something that looks good. I think data mining is really dangerous when done with a vast amount of data (overfitting I guess) so I'll refrain from diving any further into the data. I may come back to it in the future so I wrote down a few suggestions:

1. Increase sample size
2. Higher threshold of conversion rate (Goals per possession) for a team to be selected and add a defense parameter (teams which concede less may be selected)
3. Possession has many definitions, test it with other possession stats.


