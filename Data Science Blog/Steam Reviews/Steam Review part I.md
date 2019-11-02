# Steam's Broken Review System 

Steam's review system is pretty bad. If you don't know, Steam is an online store for buying mostly computer 
games (and some some software as well). Their review (and game ranking) system works like this: A player can give a game a positive or negative review. All games get an _Overall Reviews_, the percentage of positive reviews since the game was released; and games older than 30 days also get a _Recent Reviews_, the percentage of positive reviews in the past 30 days.

There are a few problems here, one is that it's pretty hard to give nuance to a review if you're forced to give a +/- rather than a 5-star system like Amazon or Google Play have. Of course even with a wider review system a large problem remains in the number of people who've voted. A game with 100% positive reviews looks impressive but will almost always have not too many reviews, as the more reviews a game or product gets the greater chance a less than perfect review will be given.

This is a less than perfect system, and plenty of people have put in their pitch for fixing the system. My personal favorite using [Evan Miller's method] (http://www.evanmiller.org/how-not-to-sort-by-average-rating.html), a self correcting ranking system based on Wilson's confidence interval. This fixes the rating system as a whole, but I'd like to propose my own fix for my own personal pet-peeve in she Steam system: the _Recent Reviews_ ranking. 

Similar to above, the biggest problem with the _Recent Reviews_ is the small sample size that makes a confident ranking less powerful when too few people vote, but where there is now only 30 days to get a good sample size. In theory the _Recent Reviews_ are supposed to give a buyer an idea of the current state of the game, a lower rating than the _Overall Reviews_ meaning the game has gotten worse through Dev activity or updates, and a higher rating meaning the game has gotten better. In practice this is almost completely useless when there are only a few reviews in the past 30 days. 

## How to Fix This:
Before going into the code and my own theory, I'll give the **tl;dr**: Only provide _Recent Reviews_ when there is enough power to be confident that the change is significant. This will help to ensure that games won't be screwed by a handful of reviewers in the past month, and the _Recent Reviews_ rating will actually serve its intended purpose.



