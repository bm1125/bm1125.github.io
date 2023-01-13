---
layout: post
title: Manhattan 2-1-2 betting system
categories: casino roulette betting 2-1-2 manhattan strategy
---

I'm going to explore an interesting strategy suggested by Aaron Brown on one of his quora [posts](https://www.quora.com/All-roulette-bets-have-a-negative-expected-value-so-what-do-roulette-systems-aim-to-achieve/answer/Aaron-Brown-165).
At the end of the day any game in the casino is negative EV game (from the bettor side) but there is a way to make things a little bit more interesting.

Aaron suggests the 2-1-2 system. According to his post the system can work in many games where the casino margin isn't too high and the payout is 2 to 1. The system is pretty simple. You first have to set the total amount of money you are willing to bet on and then divide it by 100. The result is your unit stake.

You start with 2 units stake. If you win the first round, set your stake to one unit and then increase your stake one unit everytime you win. Everytime you lose, you set your stake to 2 units.

Below is a code I wrote to simulate it. *N* is the number of simulations to run.

```python
	def manhattan_212(N):
    
        sim = np.zeros((N,115))
        
        for k in range(N):
            
            bankroll = 100
            stake = 2
            start = True
            bank_hist = [bankroll]

            for i in range(114):

                roulette_result = np.random.choice([0,1], p = [0.514, 0.486])

                if roulette_result == 1: #I Won
                    if start == True:
                        stake = 1
                        start = False
                    else:
                        stake += 1
                    bankroll += stake*2-stake
                    
                else:
                    bankroll -= stake
                    start = True
                    stake = 2
                bank_hist.append(bankroll)
                
            sim[:][k] = bank_hist
        return sim
```

I've plotted the simulation results:

![manhattan-2-1-2-betting-strategy](/assets/manhattan-2-1-2.png)

According to the simulation, the strategy will yeild a non-negative return rate in approx 33% of the times.  ~18.5% of the simulations yielded more than 15% return and, ~3% of simulations had a return rate of over 50%! None of the simulations went down to 0. One had a return rate of over 100%.

I wanted to compare the simulation to the no-strategy of just betting the same amount every round. 

```python
	def same_bet(N):
    
        sim = np.zeros((N,115))
        
        for k in range(N):
            
            bankroll = 100
            stake = 1
            bank_hist = [bankroll]
            
            for i in range(114):
                
                result = np.random.choice([0,1], p = [0.514, 0.486])
                
                if result == 1:
                    
                    bankroll += (stake*2-stake)
                
                else:
                    
                    bankroll -= stake
                
                bank_hist.append(bankroll)
            
            sim[:][k] = bank_hist
        return sim
```

The regular strategy had actually more success rate. with ~41% of the simulations ended with non-negative returns, ~4% with over 15% and the highest return rate was 28% (compared to 131% on 2-1-2).

Now to the plot of the end results (bankroll balance in unit stakes after the last round of roulette) of 1000 simulations of both strategies:

![manhattan-2-1-2-vs-regular-bets](/assets/sim-results.png)

So I guess the average outcome of the non-strategy is actually better, but we can see the heavy tails of the 2-1-2 strategy. Now I think the 2-1-2 strategy biggest advantage is that it is much more exciting than regular betting. Think about adding up to your stake everytime you win compared to having the same stake over and over again.
Besides, if I'm going to the casino I am not opting for an average result.
