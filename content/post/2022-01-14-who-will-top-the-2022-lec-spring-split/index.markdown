---
title: Who will top the 2022 LEC Spring Split?
author: Ben Vigreux
date: '2022-01-14'
slug: who-will-top-the-2022-lec-spring-split
categories:
  - prediction
tags:
  - random-forest
  - league-of-legends
---

## tl;dr 

I build a simple predictive model to predict which teams will do well in the 2022 Spring Split of the League of Legends European Championship. The random forest model is based entirely on average player performance over the last 5 games and will serve as a baseline for future attempts! See how these predictions compare to latest game results [here!](https://rdvark.shinyapps.io/LoL-LEC-Predictions-Tracker/?_ga=2.63722605.355509868.1643653815-135412473.1643148085)

## We live in exciting times

Today is a momentous day. Today marks the start of the 2022 Spring Split of the League of Legends (LoL) European Championship (LEC). You hadn't heard?! How?! 

Although I haven't played LoL in years, one of my guilty pleasures is to follow the LEC every year and cheer for my favourite teams. There is nothing quite like sitting on the couch, watching 10 teenage boys (let's face it, professional 'esports' generally have a big gender problem!) frantically hitting their keyboards and mice in skillful ways that mere mortals could only dream of. In this day-and-age where data is king, I naturally wonder, could I build a model to successfully predict whether my favourite teams will do well this year? So I came up with a simple model to get some quick predictions in before the start of the Split. 

## What is League of Legends

For the uninitiated, LoL is a computer game in which two teams (red vs. blue) of 5 players face off on _Summoner's Rift_ (represented below). The goal of the game is to destroy the enemy team's _Nexus_. To do this, a team has to play together and use their abilities/spells to kill the enemy team's players; break the enemy's towers to make a path to the Nexus; and hit the damn thing until it explodes. 

#### A simple representation of Summoner's Rift. Pentagons represent each team's Nexus; and circles their towers. Adapted from [EsportsTalk](https://www.esportstalk.com/blog/eli5-league-of-legends-guide-23576/).
![Simple representation of Summoner's Rift](images/summoners-rift.png)
During the first phase of the game, each player in a team generally has a specific role: top-laner, jungler, mid-laner, bot-laner or support. 
- Top laners face each other in the top lane; 
- mid-laners in the middle lane;
- bot-laners in the bottom lane (so far so good!); 
- supports help their bot-laners so follow them in the bottom lane;
- junglers hang around in their jungle and occasionally help any/all of their laners.

## What is the LoL European Championship?

The LEC is the professional league in which the top 10 European teams compete to win the title of European Champions (along with the prize money and bragging rights that come with). The league is divided into two _Splits_, Spring and Summer. Each Split is, well... split... into two stages: 
- a double [round-robin](https://en.wikipedia.org/wiki/Round-robin_tournament) competition in which each team plays against all other opponents twice (each team plays a total of 18 games); and 
- a tournament that determines the European champions. 

For simplicity, and because it's generally easier to predict, the rest of this post focuses solely on predicting the double round-robin stage, which starts today! 

Feeling lost already? Fear not, this was by far the hardest bit.

## What's the data like?

A nice aspect of esports is that there is a lot of data that is generated when two teams play and that data is readily digitised. Thanks to the folks at [Oracle's Elixir](https://oracleselixir.com/), data from all professional LoL competitions since 2014 is available for anyone to download (though data for earlier years is sparser)! This data includes metrics such as how many kills each player secured during the game, how many times they died to the enemy team, the amount of damage done to enemy champions, and crucially, whether their team won the game. 

For my predictions, I'll be using data:
- From 2017-2021
- From leagues in which current LEC players are playing/have played (I'm not too interested in what happened in other leagues for now, although we may want to expand our model to more leagues in the future to see if it helps with our predictions!)
- From leagues with a similar round-robin format  (I do my best to exclude tournaments for example)
- About individual players' performance. There have been CRAZY amounts of [changes in the LEC team rosters](https://lol.fandom.com/wiki/Roster_Swaps/Current/Europe) since the last season. So much so that each team is virtually new. As a result, I think it's best if we don't make predictions based on previous team performance and focus on player performance.


## Averages? Averages!

For each team-game combination in the data, I calculate each participating player's average performance across 64 player metrics in  their previous 5 games. Why 5? It's completely arbitrary; but it does have some interpretability as it's roughly a quarter of a split. Once missing observations are dropped, this leaves me with approx. 15.8k team-game combinations.  

## Let's grow some trees!

I then build and tune a classification model, which predicts whether a team won or lost based on the average performance of its players in their previous 5 games. The specific type of model I use is a [random forest](https://en.wikipedia.org/wiki/Random_forest). 

I LOVE random forests. Not just because of their names; but because they:
- are generally decent predictors;
- can be used for both regression and classification problems; 
- are flexible / not reliant on as rigid assumptions as, say, linear models; and
- are based on decision trees, which are interpretable.

One particular advantage of random forests is that they are not as vulnerable to collinearity between variables (although when it comes to collinearity, [they are not completely out of the woods yet](https://stats.stackexchange.com/questions/141619/wont-highly-correlated-variables-in-random-forest-distort-accuracy-and-feature)(sorry!)). With 321 predictors in my data (64 for each of team's 5 players, and one indicator of whether the team played on blue/red side), many which could be correlated, and the fact that I needed to rush to get this out in time for the start of the split, that appealed to me!

After a bit of hyperparameter tuning, the model I end up with is a model which __accurately predicts 58% of games__ from a held-out test set; and with an __[Area Under the Curve](https://towardsdatascience.com/understanding-auc-roc-curve-68b2303cc9c5) of 61%__ (ROC Curve below). 

#### ROC curve for the baseline model.
![ROC Curve of the baseline model](images/baseline_model.png)

## Now for my predictions!

Now that I have a model, I need to construct a matrix of the new LEC teams and their players' performance in the previous 5 games. For most of the 50 players this is straightforward, except for:
- Astralis player Dajor, who it appears hasn't yet played 5 games in professional leagues yet. 
- SK Gaming player Treatz, who is expected to play the role of support but played the role of jungler in his previous 5 games. 
- MAD Lions player Reeker, who is missing data for 2 variables. 

I replace all these missing values with the average performance of other players in the same role. 

With the matrix in hand, I can now use the model I built to predict each new LEC team's likelihood of winning. Assuming, each team will play each opponent once on blue side and once on red side, we can compare teams' predictions of winning a red-side/blue-side game. The following are 15 of our 90 matchup predictions.


```
## # A tibble: 15 x 5
##    blue_side blue_win_pred red_side red_win_pred winner  
##    <chr>     <chr>         <chr>    <chr>        <chr>   
##  1 astralis  55.44%        bds      51.61%       astralis
##  2 astralis  55.44%        excel    43.99%       astralis
##  3 astralis  55.44%        fnatic   58.52%       fnatic  
##  4 astralis  55.44%        g2       58.13%       g2      
##  5 astralis  55.44%        madlions 55.93%       madlions
##  6 astralis  55.44%        misfits  52.08%       astralis
##  7 astralis  55.44%        rogue    58.09%       rogue   
##  8 astralis  55.44%        skgaming 42.04%       astralis
##  9 astralis  55.44%        vitality 53.23%       astralis
## 10 bds       53.66%        astralis 55.39%       astralis
## 11 bds       53.66%        excel    43.99%       bds     
## 12 bds       53.66%        fnatic   58.52%       fnatic  
## 13 bds       53.66%        g2       58.13%       g2      
## 14 bds       53.66%        madlions 55.93%       madlions
## 15 bds       53.66%        misfits  52.08%       bds
```

We can then summarise the predicted winners across the entire Split and come up with the following predicted league table:


```
## # A tibble: 10 x 3
##    predicted_winner predicted_wins predicted_losses
##    <chr>                     <dbl>            <dbl>
##  1 fnatic                       16                2
##  2 g2                           16                2
##  3 rogue                        16                2
##  4 madlions                     12                6
##  5 astralis                     10                8
##  6 bds                           7               11
##  7 vitality                      7               11
##  8 misfits                       4               14
##  9 excel                         1               17
## 10 skgaming                      1               17
```

We're predicting a 3-way tie! Upon first glance and based on this year's new rosters, the top of the table looks plausible to me. I do think that some teams are underrated by the model though. I think MAD Lions will do even better than fourth place. Vitality is also stacked with talent this year, and I expect they will do better than this model predicts. 

In any case, in addition to enjoying the games this Split, I will be crossing my fingers hoping that this model correctly predict at least 58% of the games!

Interested in seeing how these predictions compare to actual game results? Look [here!](https://rdvark.shinyapps.io/LoL-LEC-Predictions-Tracker/?_ga=2.63722605.355509868.1643653815-135412473.1643148085)

## Parting thoughts

Now, a predicted accuracy of 58% is not amazing, but it will do for now. This first model is intended as a baseline model. In the future, I will try and produce smarter models to improve predictive accuracy. 

There are many avenues to find improvements. In no particular order:
- Building new/better features (very little feature selection or feature engineering has gone into the baseline model); 
- Considering player and/or team history in head-to-heads with specifc opponents;
- Considering which champions are considered to be strong (_meta_) at the moment and players' proficiency with strong champions;
- Increasing the amount of data by including more leagues and/or non-professional games; 
- Looking much further than players' last games;
- Steering away from averages and weighting recent games higher than older games;
- And the list goes on...


