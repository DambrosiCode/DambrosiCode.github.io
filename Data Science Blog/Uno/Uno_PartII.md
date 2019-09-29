## Trying Some Strategies

In my last post, I made a small Uno simulation in R in the hopes of finding a play-strateegy that would allow me to crush my family. By tinkering with the code I'll pit
the single strategy against 3 other players who will play in the normal fasion. I've picked out a few strategies I'd like to try:
1) Numbers First
	Here the player will prioritize playing numbers over colors
2) Play Suits First
	Here the players will prioritise playing suits over numbers
3)Pick Random Wild Color
	Player will pick a color regardless of if it's in their hand or not
4) Pick Wild Color from Most Common Color in Hand
	Here the player, instead of picking a random color, will pick the most common color in their hand
5) Play Wilds Last
	Here the wild cards will be played if there are only wild cards left in the hand. This is a strategy I've personally tried and anecdotally let me say, I'm not too confident. 
	
For reference: the normal strategy is to play a random matching card. If there's no match then to play a wild, and choose a color for the wild from a color in-hand. If the 
player can play nothing they take one card and move on to the next player. My family doesn't play taking extra cards, stacking, or points, so I won't consider them here. 
	
## What I'm Doing

As mentioned, I'm simplying going to tweak the Play() function for one player so they play it a bit different than the other players. Since I have a family of 4 I figure one 
strategizing, and 3 playing "normal", will be a good test. I'll run 1000 games in a for loop that looks like this 
INSERT CODE HERE
This will make a data frame with 1000 rows (games) and two columns (Winner, Turns Taken). I'll always be using Player 1 as the strat-tester. Finally, the actual Play()
function is more than a few lines of code, and I'll only be changing a line or two, so I won't be posting the code and only the results here. I'll explain the changes and 
for the morbidly curious, you can find the code here. INSERT LINK TO GITHUB PAGE

## Strategies

### No Strategies  
First I need to establish a baseline. Hypothetically, the players should each have a 25% chance of winning, but there may be a bias towards player turn which may a be a strateegy
in and of itself. I also need a control to compare the actual strategy win/loses against, therefore I'll run n sessions where all 4 players play normally.

To determine the session iterations I need to calculate an adequate sample size. Using a CI of .05, we'll want a precision of .025. We're assuming that there's a Pr{win = .25} so using the sample.size.prop() function in the samplingbook library in R we can calculate 
``` 
samplingbook::sample.size.prop(e = .025, P=.25)
```
we need to run approximately 1153 games.

![No Strategy strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/No Strategy.png)

It looks pretty much like you'd expect. Of course there is some variance so it's not a perfect 25/25/25/25, but considering there's nothing weird here I'll move onward, using a .25 probability of winning as a baseline for binomial test. 

### Play Suits First  
Matching in a game of Uno can involve suits or symbols, however the chances of getting a matching symbol is much smaller than a suit, and usually only done to change
a color without a Wild Card. Therefor there may be some benefit choosing to play suits over symbols, or vice-versa. We'll look at both. 

![Suits Priority strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/Suit Priority.png)

Not much of a change from playing no strategies. 

### Play Symbols First
Now we'll try it the other way

![Symbol Priority strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/Symbol Priority.png)

Again, not much of a boon or bust. It may offer a slight disadvantage when it changes the color to something you don't have in hand, but otherwise it won't help.

### Pick a Random Wild Color
Generally speaking when a player picks a wild card color they like to pick from a color in their hand to increase their odds of being able to go
in the next round. However, it's entirely possible that the color in play will change by the time it gets back around to that player, not only negating his color
choice but also increasing the odds that there is now a color he CAN'T play. 

![Random Wild Color strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/Random Wild.png)

Interstingly choosing a random wild doesn't seem to hurt or help an uno game in any statistically meaningful way. However, it may be useful in a real game witth humans who strategize like the world poker tournament. Choosing a random color clearly doesn't hurt one's chances of winning, but it can help to conceal what colors one has in their hand, thereby giving a slight edge.

### Pick Wild Color from Most Common Color in Hand  
For the same reasons as above we'll see what happens when the color choice is more calculated.

![Most Common Wild Color strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/Most Wild.png)

It's pretty clear here that there's not a real benefit to picking a wild card from a color in the hand, likely for the same reason it doesn't hurt to choose a random color.

### Play Wilds Last
Here's the most common strategy I've heard other people play. By saving Wild Cards for the last you're guarenteed to win as long as no one else +4 or +2's you 
and doesn't win before you. However, it may be a detriment to save wild cards since that means you'll be drawing cards when you could be playing.

![Wild Last strategy]({{site.url}}{{site.baseurl}}/Data Science Blog/Uno/Wild Last.png)

Finally, it looks like playing wilds last is the worst strategy on the board here. Reducing the winning rate by nearly 60%, there's no reason anyone should do this unless they want to lose.

## Conclusion
As far as Uno strategies go using brute statistics, there aren't many. I, of course, may be overlooking some (and please let me know if I am) but from what I can gather the game is as random as it looks, at least when playing to a strategy. But keep in mind this is a game with humans and human rationality and it's impossible to code that kind of thought process into a simulation such as this. However psyching out your opponent, calling them out for not saying "Uno" and yes, even flipping over the table in bling rage after your THIRD PLUS FOUR WILD CARD are all valid strategies I couldn't account for. 

As far as future strategies based off of this the only one that looks feasible is to randomly choose a wild card color to keep opponents guessing, and never save wilds until the last card. 








