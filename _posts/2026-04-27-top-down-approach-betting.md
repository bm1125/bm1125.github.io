---
layout: post
title: Top down approach that may work
usemathjax: true
categories: [Football, Betting, Sport]
---

I have spent some time trying a [alert service](https://www.pinnacleoddsdropper.com) for pinnacle odds. The idea is that once odds dropped on pinnacle (and assuming after limits have been raised and we are close enough to kick off so no new game changer information is expected), one can find other bookmakers to place a bet at the pre-changed odds and it is expected to be a +EV bet. The results so far look okay although it is defnitely too early to tell if I am doing it correctly. total of 132 bets with $832 turnover and $55 profit (amounts to %6.65 yeild).

There are many issues with this approach. What I have noticed so far, is that for bigger leagues, most bookmakers I follow seems to use some service for odd feeds so they will be kept updated with pinnacle odds. There is usually just few seconds delay between the bookmaker updating the odds and the service I paid for. For many other bookmakers, latency seemed to be zero. Once I get alert, they are updating their lines as well. 

After trying it for a while, I noticed something interesting. It seems to me that on big match days, the bookmakers are quick to update lines with accordance of pinnacle lines. But on some "dead" days, ie weekdays, and for some lower leagues, some bookmakers seem to get complancent and let 1X2 or over/under lines stay outdated. Now I am talking about lower leagues, on midday, the VIG for these kind of leagues are already high (> 7%) and the error is rarely more than 3% so it's not enough to surpass the vig in itself (ie finding home team mispriced by 2-3% not enough to overcome vig of 7-9%).

A 1X2 error indicates that one of the model parameter is off. As I explained in my previous post, the 1X2 odds (along with the AH) are what sets the the $$\ \lambda_{home}, \lambda_{away} $$ and if the odds are off it means the parameters are off. This error will then trickle down to the BTTS line calculation. The magnitude of the error could be different on the BTTS but that doesn't matter as it is a secondary market hence it has even a higher VIG so even with the error I could not find any positive expected bets there.

But then there is a special combo bet of 1X2 & BTTS. Since it is suppose to be kind of a paralay bet, the error magnitude may be big enough to overcome the VIG and produce +EV bet. In order to be able to take advantage of it, I used chatgpt to build a calculator to estimate parameters ($$\ \lambda_{home}, \lambda_{away}, \rho $$) of the model and using these parameters to estimate specifically each possibility of the 1X2 & BTTS bets.

The screenshow below will show the odds found at some online bookmaker along with the odds offered by pinnacle according to pinnacleoddsdropper service. The home team is priced at pinnacle at 1.8 while the bookmaker still offers 1.88 this amounts to 2.3% absolute different. From my experience, around 2% is the bare minimum needed to find +EV. Important to note this is 15 minutes to kick off, so overall big changes to the lines are not expected.

![44vave](/assets/top-down/1X2.png)
![pinnacle_odds](/assets/top-down/pod_screenshot.png)

The first column in the second screenshot, from the pinnacle odds dropper service, is the no vig price, computed using the power method. Using 1X2, over/under and AH I could estimate the model parameters as shown in the screenshot below. To verify myself I check that my clean pick side estimation (in this case home) is same as the no vig price from the service I used. We have 1.872 from the service and 1.8702 from my calculator.

![poisson_estimation](/assets/top-down/poisson_calculator.png)

So far, from my experience, it is not that all home options are going to be +EV, but rather only one, and even then not necessarily. I am not entirely sure why in some cases there is a big divergence (> 2%) in 1X2 prices between the bookmaker and pinnalce, and still none of the combo bets are estimated to be +EV. In this example, see the Home & BTTS No which its fair price estimated to be 2.416. Checking the bookmaker price, I found 2.95 which makes it +EV.

![bookmaker_combo_1X2_btts](/assets/top-down/bets.png)  

I don't have any conclusive results as the sample size it too small for now (40 bets). I doubt that I will keep doing it as the it is not scaleable. The limits depend on the odds, and for odds around 3 bets are limited to around $11. Overall it is profitable but again, sample size way to small to conclude anything.