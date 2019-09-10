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
5) Pick Wild Color from Least Common Colot in Hand
	Here the player, instead of picking a random color, will pick the least common color in their hand
6) Play Wilds Last
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
in and of itself. I also need a control to compare the actual strategy win/loses against, therefore I'll run 1000 sessions where all 4 players play normally.
INSERT GRAPH

### Play Suits First  
Matching in a game of Uno can involve suits or symbols, however the chances of getting a matching symbol is much smaller than a suit, and usually only done to change
a color without a Wild Card. Therefor there may be some benefit choosing to play suits over symbols, or vice-versa. We'll look at both. 
INSERT GRAPH

### Play Symbols First
Now we'll try it the other way
INSERT GRAPH

### Pick a Random Wild Color
Generally speaking when a player picks a wild card color they like to pick from a color in their hand to increase their odds of being able to go
in the next round. However, it's entirely possible that the color in play will change by the time it gets back around to that player, not only negating his color
choice but also increasing the odds that there is now a color he CAN'T play. 
INSERT GRAPH

### Pick Wild Color from Most Common Color in Hand  
For the same reasons as above we'll see what happens when the color choice is more calculated.
INSERT GRAPH

### Pick Wild Color from Least Common Color in Hand
And just one more instance
INSERT GRAPH

### Play Wilds Last
Here's the most common strategy I've heard other people play. By saving Wild Cards for the last you're guarenteed to win as long as no one else +4 or +2's you 
and doesn't win before you. However, it may be a detriment to save wild cards since that means you'll be drawing cards when you could be playing. 
INSERT GRAPH









