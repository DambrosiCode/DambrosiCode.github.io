## Family Rivalry
My family has always been into board and card games so everytime I 
go home we usually play a few games, usually one more than another. Recently this was 
the classic card game/yell at the guy next to you for screwing you over Uno. After several rounds,
many which went on for what felt longer the life of the Sun, I had learned one thing: I suck at Uno.

I don't know how this happens, there's really not much of a strategy you can employ, since it's pretty
reactionary. Or is it? I had a few ideas for a strategy and not nearly enough time to play it.
So I decided to make my computer do all the work. 

## Making a Game
Uno is a pretty simple game when you get right down to it. It follows an easy progression of 
steps and making a simulation that plays itself a few thousand times shouldn't be too hard. 
When I was first learning Python a few years ago I had made Uno's blood relative, [War]() but
this time I decided to use R. **DISCLAIMER**: If you're running iterative simlations I'd recommend Python 
since the snytax is a bit cleaner and it's much faster than R. I chose to use R for no other reason
than I had never made a card game simulation in it.

### The Process
Uno is a straight-forward game that can be broken down in several parts:
1) Setup (once)
2) Place matching card
3) Draw (optional)
4) Uno (optional)
5) Win (optional)

There's a few rules for how drawing works that varies between people and households and the 
actual official rules. I decided to go with the house rules which are:
1) No stacking + cards
2) Draw one card if you can't match and move on 
3) No points, you either win or lose
Additionally there's no real way for me to make the computer "forget" to say Uno and take +4
unless since some people are more astute about these things than others. So I assumed players
would always say "Uno" as soon as they had one card in their hand. 

### The Code
Now it was just time to put these game parts into machine code.
#### Global Variables
I essentially wanted data frames to act as player hands, with a column for the 'symbol'(number, wild, or action)
and a column for the 'suit' (color). These global variables could then be put into a function
that would play a "card" in it or add another card/cards from the deck.

I also made a data frame for the "deck" (I called 'new.deck') that and "discard" that was essentially a large hand, 
where the deck would be populated with all the cards, and discard would be populated with played cards 
that would be put back into the deck when the deck <= 0. 

#### draw()
The all important Draw() function works by using rbind to take combine two data frames. In
this case the two data frames are the hand and the deck. The 'cards' parameter takes an integer
which it will randomly pull rows from the new.deck data frame. This keeps me from having to
"shuffle" the deck everytime it's put back into play.  
```R
draw <- function(cards, deck){
  if (cards > nrow(new.deck)) {
    new.deck <<- rbind(new.deck, discard[1:nrow(discard)-1,])
  } #reshuffle the discard into the drawing deck
  
    draw <- c(sample(nrow(new.deck), cards))
    cards.drawn <- new.deck[draw, ] 
    new.deck <<- new.deck[-draw,]
    return(rbind(deck, cards.drawn))
}
```

### next.turn()
Since I wanted to keep track of players by assigning them numbers, I had to make a quick
function that would loop through 1:n restarting at 1 if i>n if the "table" was going "clockwise"
and restarting at n if i<1 if the table was "counterclockwise" (IE a reverse card was played)
```R
next.turn <- function(player){

  current.turn <- player.turn + next.player #move to next player
  
  if (current.turn > length(player)) {     #return to player 1
    current.turn <- 1
  } 
  else if (current.turn < 1) #return to last player
  {
    current.turn <- length(player)
  }
  
  return(current.turn)
}
```
### setup()
Next I needed a function that would setup the table. In this case it would need to do
several things: generate a deck, deal cards (7 per player), set up a card to play that 
players must follow. 
```R
setup <- function(){
  player <<- list()
  
  #for player turn
  next.player <<- 1 #clockwise is 1, counter clockwise is -1
  player.turn <<- 1
  
  #generate deck
  symbol <- c(0, rep(c(1:9, 'draw', 'skip', 'reverse'), 2))
  suit <<- c('red', 'blue', 'green', 'yellow')
  wild <- c(rep(c('wild', 'wild.4'),4)) %>%
    data.frame(symbol = ., suit = .)
  
  new.deck <<- list(symbol = symbol, suit = suit) %>%
    expand.grid(.) %>%
    rbind(., wild)
  
  #play pile
  discard <<- data.frame()
  
  target.card <<- data.frame()
  target.card <<- draw(1, target.card) #card players must follow
}
```
### check.win()
Of course something needed to tell the compuer when a player had won. Here, if the player
has less than 1 card in their hand it will change the target.card variable to 'win' and return
the winning player's number.
```R
check.win <- function(player){  
  if(nrow(player) == 0 | is.null(player)){
    target.card <<- 'win'
  return(list(player, 'YOU HAVE WON'))
  }
}
```
### play()
I needed a function to play all the other functions in a logical manner. This function essentially
takes a 'turn' for the player after a game is 
setup().

A player's hand is checked (remember players are data frames) for symbols or suits
that are the same as the target.card's symbol or suit. This is only including non-wild cards
because those will be dealt differently. If it finds a match it will remove that card from 
the hand and check to see if the player has won. If there isn't a match the function looks
for a wild or wild.4 card which it will then play and then change it's color to a color
in the player's hand that the next player must match. Finally, if there is no match whatsoever
the player will draw and end their turn. After each play the function checks to see if the player
has won or not. 
```R
play <- function(player){
  player.hand <- data.frame()
  #find matching card
  match <- which(player$suit %in% target.card$suit & !player$symbol == 'wild' & !player$symbol == 'wild.4'
                 | player$symbol %in% target.card$symbol & !player$symbol == 'wild' & !player$symbol == 'wild') #make sure the player isn't choosing wild cards
                                                                                                                    #because the symbol never changes, check for 'wild' symbol
  
  wild <- which(player$symbol == 'wild' | player$symbol == 'wild.4')
  #remove matching card from hand
  if (length(match) > 0) {
    #print("match")
    chosen.card <- sample(match, 1)
    player.hand <- player[-chosen.card,]
    target.card <<- player[chosen.card,]
    discard <<- rbind(target.card, discard)

    check.win(player)
    
    return(list(player.hand, target.card))
  } #only use wild if it's the last possible move
  else if(length(wild) > 0){
    #print("wild")
    chosen.card <- sample(wild, 1)
    player.hand <- player[-chosen.card,]
    
    if (length(match) > 0) {
      non.wild.suit <- player$suit[player$suit != 'wild' &
                                     player$suit != 'wild.4']
    } 
    else {
      non.wild.suit <- sample(suit, 1)
    }
    
    #print(non.wild.suit)
    
    target.card <<- data.frame(symbol = player[chosen.card,]$symbol, 
                               suit = sample(non.wild.suit, 1)) #suit is randomly chosen from hand
    
    discard <<- rbind(target.card, discard)
    
    check.win(player)
    
    return(list(player.hand, target.card))
  } 
  else {
    #print("draw")
    player.draw <- list(draw(1, player))
    
    check.win(player)
    
    return(player.draw)
  } 
}

```


