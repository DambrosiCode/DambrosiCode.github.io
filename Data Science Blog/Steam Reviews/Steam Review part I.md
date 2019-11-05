# Steam's Broken Review System 

Steam's review system is pretty bad. If you don't know, Steam is an online store for buying mostly computer 
games (and some some software as well). Their review (and game ranking) system works like this: A player can give a game a positive or negative review. All games get an _Overall Reviews_, the percentage of positive reviews since the game was released; and games older than 30 days also get a _Recent Reviews_, the percentage of positive reviews in the past 30 days.

There are a few problems here, one is that it's pretty hard to give nuance to a review if you're forced to give a +/- rather than a 5-star system like Amazon or Google Play have. Of course even with a wider review system a large problem remains in the number of people who've voted. A game with 100% positive reviews looks impressive but will almost always have not too many reviews, as the more reviews a game or product gets the greater chance a less than perfect review will be given.

This is a less than perfect system, and plenty of people have put in their pitch for fixing the system. My personal favorite using [Evan Miller's method](http://www.evanmiller.org/how-not-to-sort-by-average-rating.html), a self correcting ranking system based on Wilson's confidence interval. This fixes the rating system as a whole, but I'd like to propose my own fix for my own personal pet-peeve in she Steam system: the _Recent Reviews_ ranking. 

Similar to above, the biggest problem with the _Recent Reviews_ is the small sample size that makes a confident ranking less powerful when too few people vote, but where there is now only 30 days to get a good sample size. In theory the _Recent Reviews_ are supposed to give a buyer an idea of the current state of the game, a lower rating than the _Overall Reviews_ meaning the game has gotten worse through Dev activity or updates, and a higher rating meaning the game has gotten better. In practice this is almost completely useless when there are only a few reviews in the past 30 days. 

## How to Fix This:
Before going into the code and my own theory, I'll give the **tl;dr**: Only provide _Recent Reviews_ when there is enough power to be confident that the change is significant. This will help to ensure that games won't be screwed by a handful of reviewers in the past month, and the _Recent Reviews_ rating will actually serve its intended purpose.

By scraping a list of Steam games off of [steamdb.info] and then collect reviews from the Steam Store, I was able to generate a dataset of a games that had more than 500 _Overall Reviews_ and also a _Recent Reviews_ rating. You can get the dataset (collected 10/29/19) and code [here]() and read a more indepth explanaition of how I scraped the data [here]().

# Looking at the data
What I'll be doing using is a simple proportion test using the prop.test() in R. The idea is simple, given a change in proportion (old reviews to recent reviews) is can we be at least 95% confident that the change is significant. This method works for what we're after because it takes into account not only the proportion shift, but also the sample size. 

Before we do this, however, we also want to make sure that the power for the test is adequate. Afterall we don't want too many instance of a significant change in reviews appearing not significant. And in this case "too many" is more than 20%, so we're looking for games that can give us at least 80% power. Unfortunately base R doesn't let us look at power the way we want to, as their power.prop.test function only allows for one sample size, and since our sample size is certainly changing I'll be using the pwr.2p2n.test() function from the "pwr" library. 

Great now everything is set up here's what the code looks like 
```
beta = 0.8
power.func <- function(data) {
  
  data <- as.numeric(data)
  
  x <- pwr.2p2n.test(n1 = data[5], n2 = data[7],
                h = ES.h(data[6], data[8]), sig.level = alpha)
    
    return(x$power)
    
reviews$Power <- apply(reviews, 1, power.func)    
}```

Using the apply()
