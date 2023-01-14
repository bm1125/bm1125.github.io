---
layout: post
title: Using EA FIFA team rankings to train a neural network
usemathjax: true
categories: [EPL, Deep-Learning, football]
---

In this post I will check if a neural network trained using FIFA team rankings could produce valuable predictions of football matches.
I actually did this quite a while ago. I just wanted to make more concise and clean notebook so I started afresh. This time I used fixtures data from football-data.co.uk (instead of football-api.org last time). I took the FIFA team rankings (defense, midfield, attack and overall) of each team in the english permier league for the seasons 17-20. 

In order to do it, I've built a web scraper to scrape data from [fifaindex.com](http://fifaindex.com). Have to be careful working with this scraper as the data on fifaindex isn't consistent. Same teams may be named differently over seasons.

I tried to predict match outcome (home win, draw, away win) and wanted to compare the probabilities of the model to betting odds probabilities

|    | Date                | HomeTeam       | AwayTeam     |   FTR |   PSH |   PSD |   PSA |   season |
|---:|:--------------------|:---------------|:-------------|------:|------:|------:|------:|---------:|
|  0 | 2017-08-11 00:00:00 | Arsenal        | Leicester    |     2 |  1.53 |  4.55 |  6.85 |       18 |
|  1 | 2017-08-12 00:00:00 | Brighton       | Man City     |     0 | 10.95 |  5.55 |  1.34 |       18 |
|  2 | 2017-08-12 00:00:00 | Chelsea        | Burnley      |     0 |  1.26 |  6.3  | 15.25 |       18 |
|  3 | 2017-08-12 00:00:00 | Crystal Palace | Huddersfield |     0 |  1.83 |  3.58 |  5.11 |       18 |
|  4 | 2017-08-12 00:00:00 | Everton        | Stoke        |     2 |  1.7  |  3.83 |  5.81 |       18 |
|...|

Using the scraper as follows:

```python

fifa = fifa_index.fifaIndex() # Initializing

fifa.setVersions(18,19,20) # Setting versions of FIFA to scrape from. 18 = 2017-2018 and so on.
fifa.scrapeLeagues(13) # See fifa.getAvailableLeagues() to see all possible leagues to scrape from.

```

If everything works fine, the output should be:

```
scraping:	 https://www.fifaindex.com/teams/fifa18/1/?league=13&;
scraping:	 https://www.fifaindex.com/teams/fifa19/1/?league=13&;
scraping:	 https://www.fifaindex.com/teams/fifa20/1/?league=13&;
```

calling `fifa.dataframe()`


Will show a final dataframe with all teams rankings for each season. The rankings is taken from the beginning of the season. I know I'll have to imporve the scraper to take more rankings throughout the season as rankings move a bit. Althought fow now, I didn't want to spend time on this as the rankings barley move throughout the season. Maybe in the future, I'll improve the scraper to include more features of the teams and more time dependent ranking.

|                           |   defense |   midfield |   attack |   overall |
|:--------------------------|----------:|-----------:|---------:|----------:|
| ('Manchester City', 18)   |        83 |         87 |       85 |        84 |
| ('Manchester City', 19)   |        82 |         88 |       86 |        85 |
| ('Manchester City', 20)   |        83 |         86 |       87 |        85 |
| ('Tottenham Hotspur', 18) |        82 |         83 |       85 |        83 |
| ('Tottenham Hotspur', 19) |        83 |         82 |       86 |        83 |
|...|


Okay so now to the model. I just set two functions so it'll be easier to test model with different parameters and playing with it. 

```python
def set_model(dropout, first_layer, second_layer):
    
    global model
    model = Sequential()
    n_features = X_train.shape[1]

    model.add(Dense(8, input_shape = (n_features,)))
    model.add(Dense(first_layer))
    model.add(Dropout(dropout))
    model.add(Dense(second_layer))
    model.add(Dense(3, activation = 'softmax'))

    opt = keras.optimizers.Adam(learning_rate=0.01)
    model.compile(loss='categorical_crossentropy', optimizer='Adam', metrics = ['accuracy'])
    
def test_model(epochs, patience):
    early_stopping = keras.callbacks.EarlyStopping(monitor='accuracy', min_delta = 0, patience = patience)
    model.fit(X_train, y_train, verbose = 0, epochs = epochs, batch_size = 1, callbacks = [early_stopping])
    return model.evaluate(X_test, y_test)
```

I really don't have experience with such models and I actually don't know what parameters to start with. So I just played with it until i've found the best parameters according to the model accuracy. I'm not sure how much emphasis I should place on model accuracy because football games are low scoring games that are much more random than other sports. It's not like a classic classification problem where you have only right answer. All options are possible, home win, draw and away win and my end goal is to create a model that will estimate the probability for each scenario, better than the betting markets.

So at the end I set the model to 2 layers of 8 and 6 nodes each and dropout rate of 0.5 and patience 15.

```python
set_model(0.5, 8,6)
test_model(1000,15)
```

```
4/4 [==============================] - 0s 875us/step - loss: 1.0003 - accuracy: 0.5175
[324]:
[1.0003230571746826, 0.5175438523292542]
```

And finally, to the results

|     | Date                | HomeTeam       | AwayTeam    | FTR   |      PSH |      PSD |       PSA |     Home |     Draw |      Away |
|----:|:--------------------|:---------------|:------------|:------|---------:|---------:|----------:|---------:|---------:|----------:|
|   8 | 2017-08-13 00:00:00 | Man United     | West Ham    | H     | 0.75188  | 0.176056 | 0.0921659 | 0.718746 | 0.196215 | 0.0850393 |
|  11 | 2017-08-19 00:00:00 | Burnley        | West Brom   | A     | 0.378788 | 0.315457 | 0.325733  | 0.533322 | 0.244876 | 0.221802  |
|  41 | 2017-09-16 00:00:00 | Crystal Palace | Southampton | A     | 0.334448 | 0.301205 | 0.384615  | 0.539307 | 0.24389  | 0.216803  |
|  62 | 2017-09-30 00:00:00 | Huddersfield   | Tottenham   | A     | 0.104822 | 0.2      | 0.714286  | 0.197168 | 0.20512  | 0.597712  |
|  76 | 2017-10-14 00:00:00 | Watford        | Arsenal     | H     | 0.182149 | 0.228833 | 0.609756  | 0.343878 | 0.243894 | 0.412228  |
|  89 | 2017-10-22 00:00:00 | Tottenham      | Liverpool   | H     | 0.45045  | 0.276243 | 0.294118  | 0.574535 | 0.238306 | 0.187159  |
| 114 | 2017-11-18 00:00:00 | Leicester      | Man City    | A     | 0.110375 | 0.169779 | 0.740741  | 0.274902 | 0.230783 | 0.494315  |
| 121 | 2017-11-25 00:00:00 | Crystal Palace | Stoke       | H     | 0.480769 | 0.290698 | 0.248139  | 0.556209 | 0.241376 | 0.202415  |
| 129 | 2017-11-26 00:00:00 | Southampton    | Everton     | H     | 0.531915 | 0.286533 | 0.200803  | 0.429639 | 0.250662 | 0.319699  |
| 139 | 2017-11-29 00:00:00 | Stoke          | Liverpool   | A     | 0.155039 | 0.21322  | 0.653595  | 0.299217 | 0.236409 | 0.464374  |
|...|


In order to test the model, I decided to check it against betting odds. In the table above there are three columns; PSH, PSD, PSA which is implied probabilities given by pinnacle odds. Now these were not derived from closing line odds but from midweek (few days before each match). I saved the results dataframe to a csv file and opened it in excel. It's much more exhausing doing it in excel but I wanted to see the model output with my eyes and not just automate everything. I looked at each and every pick the model had. The results were quite surprising to be fair. Total of 114 bets with average odds of 3.784 and 28.5% return! Using [this](https://www.football-data.co.uk/blog/P-value_calculator.xlsx) p value calculator I saw the probability of the model actual being profitable (than just being profitable due to pure chance) is 4.9% .

![fifa-model-betting-results](/assets/fifa-model/betting_results.png)

A full notebook is available on [github](https://github.com/bm1125/fifa/blob/master/FIFA.ipynb).