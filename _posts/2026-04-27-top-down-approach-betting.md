---
layout: post
title: Top down approach that may work
usemathjax: true
categories: [Football, Betting, Sport]
---

I have spent some time trying a [alert service](https://www.pinnacleoddsdropper.com) for Pinnacle odds. The idea is that once odds dropped on Pinnacle (and assuming after limits have been raised and we are close enough to kick off so no new game changer information is expected), one can find other bookmakers to place a bet at the pre-changed odds and it is expected to be a +EV bet. The results so far look okay although it is definitely too early to tell if I am doing it correctly. total of 132 bets with $832 turnover and $55 profit (amounts to %6.65 yield).

There are issues with this approach. What I have noticed so far, is that for bigger leagues, most bookmakers I followed seem to update their lines in accordance with Pinnacle odds and with a few seconds delay. For others, latency seemed to be almost zero, Once I received alert, they are updating their lines as well. 

After trying it for a while, I noticed something interesting. It looks like on some "dead" days, ie weekdays, and for lower leagues, some bookmakers get complacent and let 1X2 or over/under lines stay outdated. Now I am talking about lower leagues, on midweek days, the vig for these matches are already high (> 7%) and the error is rarely more than 3% so the 1X2 mispring (either home or away) is not enough to surpass the vig in itself.

A 1X2 error (again, either home or away mispricing) indicates that one of the model parameter is off. As I explained in my previous post, the 1X2 odds (along with the AH) are what sets the the $$\ \lambda_{home}, \lambda_{away} $$ so if the odds are off it means the parameters have not been updated. This error will then trickle down to the BTTS line calculation. The magnitude of the error could be different on the BTTS but it is a secondary market hence it has even a higher vig so even with the error I could not find any positive expected bets there.

But then there is a special combo bet of 1X2 & BTTS. Since it is suppose to be kind of a parlay bet (although they are not independent events), the error magnitude may be big enough to overcome the vig and produce +EV bet. In order to be able to take advantage of it, I used chatgpt to build a calculator to estimate parameters ($$\ \lambda_{home}, \lambda_{away}, \rho $$) of the model and using these parameters to estimate specifically each possibility of the 1X2 & BTTS bets.

The screenshot below will show the odds found at some online bookmaker along with the odds offered by Pinnacle according to Pinnacleoddsdropper service. The home team is priced at Pinnacle at 1.8 while the bookmaker still offers 1.88 this amounts to 2.3% absolute different. From my experience, around 2% is the bare minimum needed to find +EV. Important to note this is 15 minutes to kick off, so overall big changes to the lines are not expected.

![44vave](/assets/top-down/1X2.png)
The screenshot above show the odds offered by the bookmaker.

![pinnacle_odds](/assets/top-down/pod_screenshot.png)
The screenshot above show the odds as offered by Pinnacle.

The first column in the second screenshot, from the Pinnacle odds dropper service, is the no vig price, computed using the power method. Using 1X2, over/under and AH I could estimate the model parameters as shown in the screenshot below. To verify myself I check that my clean pick side estimation (in this case home) is same as the no vig price from the service I used. We have 1.872 from the service and 1.8702 from my calculator.

![poisson_estimation](/assets/top-down/poisson_calculator.png)
I use this calculator to enter odds and estimate parameters of the model (first table) and then estimate different bet combinations.

So far, from my experience, it is not that all home options are going to be +EV, but rather only one, and even then not necessarily. I am not entirely sure why in some cases there is a big divergence (> 2%) in 1X2 prices between the bookmaker and pinnalce, and still none of the combo bets are estimated to be +EV. Also, could never find a draw option of this combo with +EV even when draw from 1X2 is highly underestimated. 

In this example, see the Home & BTTS No which its fair price estimated to be 2.416. Checking the bookmaker price, I found 2.95 which makes it +EV.

![bookmaker_combo_1X2_btts](/assets/top-down/bets.png)
These are the lines offered for 1X2 & BTTS by the same bookmaker as in the first screenshot.

I have had 39 bets so the sample is to small to conclude anything. Staked $332 and profited $168.23. This is obviously not scalable so I will not pursue it anymore. The limits depend on the odds, and for odds around 3 bets are limited to around $11. There is also operative limit as I live in a country where I can't access to most betting sites.