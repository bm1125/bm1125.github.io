---
layout: post
title: Goal scoring ability over number of resting days
usemathjax: true
categories: [EPL, football, GLM]
---

I wanted to see if there is any relationship between the days of rest (or not actually rest, but days since last match) to the amount of goals a team will score in a game.
I used the data from football-data.co.uk for the english premier league season 2018-2019.

|    | Date                | Team      |   Goals |   Days |
|---:|:--------------------|:----------|--------:|-------:|
| 15 | 2018-08-18 00:00:00 | West Ham  |       1 |      6 |
| 14 | 2018-08-18 00:00:00 | Tottenham |       3 |      7 |
| 13 | 2018-08-18 00:00:00 | Leicester |       2 |      8 |
| 11 | 2018-08-18 00:00:00 | Chelsea   |       3 |      7 |
| 10 | 2018-08-18 00:00:00 | Cardiff   |       0 |      7 |
|...|

Where days is the number of days since last match of that specific team.

![epl-days-of-rest-goals](/assets/rest-days/epl_rest_plot.png)

Looking at the data, I personally can not see any clear trend out of it. Decided to run a linear regression anyway. This regression is pretty simple as I only have on independent variable.

$$ \text{Goals} \sim Pois(\lambda_i) \\
\lambda_i = \alpha + \beta * \text{Days} \\
\alpha \sim N(0, 1) \\
\beta \sim N(0, 1) 

$$

Regression summary shows the slope hdi is on both sides of the zero, meaning there is no clear trend. 

|    |   mean |    sd |   hdi_3% |   hdi_97% |
|:---|-------:|------:|---------:|----------:|
| a  |  0.375 | 0.072 |    0.249 |     0.518 |
| b  | -0.004 | 0.009 |   -0.02  |     0.013 |

To see it with our own eyes:

![glm-days-goals](/assets/rest-days/reg_results.png)

The dashed line is the estimated mean of all samples for the $$ \lambda $$ parameter and the lines above and under indicate 95% highest density interval for that parameter.

Problem with this analysis that I included resting days of over 7 days which is probably due to holiday or international break (Nations league, EURO Quanlifiers, WC Qualifiers). I wanted to see if there is any trend within 7 days only. Meaning during regular season, if I eliminate breaks, is there a different if a team is given 6 days of rest or only 3 ? I know I can not just decide to cut my data in half and run regression but I just wanted to peek if there's any trend at all that worth exploring. 

![glm-days-goals](/assets/rest-days/reg_results2.png)

So same as before, no clear direction at all. Wether a team had only three days of resting or seven, it doesn't seem to affect their ability to score goals at all.

I think what still needs to be checked is more of intensity periods. Maybe a team can play one game only three games after their last game without it affecting their ability, but what if they had to play 3,4,5 games in a period of two weeks or less ? I'll check that in the future.

