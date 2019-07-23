---
layout: default
---

## A Data Science Look at Country Lyrics

  This July I was over a friend's house for the Fourth. We went out for drinks, and my friend bemoned the bar's live country singer. I'd agreed with everything she said up until the singer dusted off the classics and I found my knowing all the lyrics to Johnny's Cash's **Ring of Fire** and Kenny Rogers's **The Gambler**. It was here that I had that surely declared not **all** country music was bad (that is unless you like all Country music, in which case forgive me), classic country seemed to be not only be good but, universally liked as much as modern is universally berated. Afterall who hasn't tapped their toes to Elvis Presely or Dolly Parton, and of course any genre with a song like **Devil Went Down to Georgia** has to have some merit. So with a long weekend, my plans were set: find out what went wrong with Country music.
  
  This is going to be a bit subjective. I'll be using cold hard data science and soulless graphs to try and look quantify something as personal as taste in music. I'll try to look at every angle, but at the end of the day, music is a personal preference and no amount of statistics or pleasantly crafted graphs can change that. But by Fisher I'll try anyway.
  
### Getting Some Data
  
  I decided that the best way to do this is to look at the top Country hits each year. I'd need to scrape some data to compile a list of Artist, Song, Lyrics, and Year produced. Luckily for me this was relatively easy thanks to the glorious, and neatly organized, website [Playback.fm](https://playback.fm/charts/country) which gave me the top 100 songs and artist for every year going back to 1944. 
  
  I decided to use Python for scraping and making my data sheet for two reasons, one: I'm more comfortable data scraping on Python, and two: the Genius library let me enter artist and song and return lyrics as a string. R can do both of these things, but everything ran just a bit smoother in Python. With a simple function (whose code you can find [here](https://github.com/DambrosiCode/music_analysis/blob/master/lyrics%20getter) ) I was able to create a csv with an Year, Artist, Song, and Lyric column in about 30 minutes. Then it was off to analysis
  
### Analyzing Some Lyrics
  
   Looking at the songs it was apparent that I'd need to focus on just the lyrics, since getting actual musical data was far beyond the scope of this project. But that would be just fine. I decided to look at three things: The amount of unique words, a sentiment analysis, and commonly words used.
   
### Something's Missing
  
  The first big hurtle I faced was the fact that more than a few songs were missing from the data. I suspected this was because Genius likely doesn't have all the lyrics for every song, and it probably would be the older songs that were less likely to be in Genius's database. And using some clever coding we can see that yes, there is about a -0.84 correlation between year and what song lyrics we can get.  
  
  
```R
library(ggplot2)
country <- read.csv('Country_Roads.txt', sep = '\t', fileEncoding="latin1")  

lost.songs <- as.data.frame(tapply(country$Lyrics, country$Year, function(x) sum(is.na(x))))  

ggplot(lost.songs, aes(y = lost.songs[,1], x = 1944:2014, color=lost.songs[,1])) +   geom_point(size = 3) + xlab('Year') + ylab('Lost Songs') + ggtitle("Lost Songs") +  theme(legend.position = "none")

cor(x = lost.songs[,1], y = 1944:2014)
```  
<img src="{{ site.baseurl }}/blob/master/Data%20Science%20Blog/images/Country%20Lyrics/Lost%20Songs%20Country.png"

![Song lyrics lost to oblivion](https://github.com/DambrosiCode/DambrosiCode.github.io/blob/master/Data%20Science%20Blog/images/Country%20Lyrics/Lost%20Songs%20Country.png)

This wasn't a huge deal since I still had plenty of data each year to work with, so I simply removed NAs and worked with what I had
```R
country <- na.omit(country)
```  

### Unique Words

To clarify, when I say "unique words" I mean the first time a new word is used, not including stop-words such as "a", "the", "and", etc... For example, "The devil went down to Georgia" would have 4 unique words in it, while "Drink a beer, drink a beer." would count as just 2 unique words.

For this I needed to make a few functions that would let me split the string into multiple strings by word, and a function that would return the number of unique words. 
```R
library(dplyr)
library(stopwords)
clean_string = function(song_lyrics){
  x <- as.character(song_lyrics) %>%
    removeWords(., c(stopwords())) %>%
    str_split(.,' ', simplify = T) %>% 
    .[. != '']
  
  return(x)
}

find_unique = function(x){
  y <- clean_string(x) %>%
    unique(.) %>%
    length(.)
  return(y)
}
```  
The first function splitting the string by word and removing stop-words, and the second one counting the number of unique words. The piping function (%>%) in also helped me to keep my sanity while reformatting.  

Then I made a new column in my dataframe that told me the unique words and voila
```R
country$Unique <- sapply(country$Lyrics, find_unique)

ggplot(country, aes(y = country$Unique, x = country$Year)) + 
  geom_point() + geom_smooth() + xlab("Year") + ylab("Unique Words") + 
  ggtitle("Unique Words in a Song", subtitle = "Country Songs")
```  
![Unique words](https://github.com/DambrosiCode/DambrosiCode.github.io/blob/master/Data%20Science%20Blog/images/Country%20Lyrics/Unique%20Country.png)
  
 It looked like there was some kind of a trend. The trend line seemed to increase with year, and using lm() we can confirm that there is about a .44 increase of unique words added to a song each year
 ```R
lm(formula = country$Unique ~ country$Year)
```  
   
I decided to average this data and see if there was a more obvious trend. 
 ```R
unique.song <- tapply(country$Unique, country$Year, mean)

ggplot(as.data.frame(unique.song), aes(y = unique.song, x = 1944:2014, color = unique.song)) + 
  geom_point(size = 2) + geom_smooth() + xlab("Year") + ylab("Unique Words") + 
  ggtitle("Unique Words in a Song", subtitle = "country Songs") + theme(legend.position = "none")
``` 
![Unique words, averaged each year](https://github.com/DambrosiCode/DambrosiCode.github.io/blob/master/Data%20Science%20Blog/images/Country%20Lyrics/Mean%20Unique%20Words%20Country.png)

As it turns out the slope seems to increase drastically somehwere in the 90s. The odd thing is, one would imagine that an increase of unique words would be a good thing for song quality, but my hypothesis was that country songs get worse over time. But once again lm() confirmed that the slope went from .16 in the first 50 years (1944-1993) to a whopping 0.825 (1995-2014). 
 ```R
lm(unique.song[1:50] ~ c(1944:1994))
lm(unique.song[51:71] ~ c(1994:2014))
``` 

Then it occured to me, the amount of unique words should be weighted against the length of a song. After all a shorter song is going to have less unique words just as a function of less space for the artist to write in. By dividing the number of unique words in a song by the number of words in general I could get a better grasp of the writer's ability to make use of the space given (a talent I clearly don't have as this article isn't close to being over). 

 ```R
library(ngram)
country$Lex <- sapply(country$Lyrics, function(x) wordcount(as.character(unique(clean_string(x))))/wordcount(as.character(x)))

country.lex <- tapply(country$Lex, country$Year, mean)

ggplot(as.data.frame(country.lex), aes(y = country.lex*100, x = 1944:2014, color = country.lex)) + 
  geom_point(size = 2) + xlab("Year") + ylab("Percent of Unique Words") + geom_smooth(se = F, method = 'lm') + 
  ggtitle("Percent of Unique Words in a Song", subtitle = "Country Songs") + theme(legend.position = "none")

lm(country.lex*100~c(1944:2014))
``` 
!(Average percent of unique words every year)[https://github.com/DambrosiCode/DambrosiCode.github.io/blob/master/Data%20Science%20Blog/images/Country%20Lyrics/percent%20unique%20country.png]

And lo and behold, when averaging over the yearlly data, there is a clear trend. Not too dramatic (about a .05% decrease every year), but it's still something, and there is more data to be sprunged. 

### Yearly Sentiments
   
[back](../../)















