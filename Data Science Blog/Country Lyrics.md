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
![Songs lost to oblivion]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Lost Songs Country.png)

This wasn't a huge deal since I still had plenty of data each year to work with, so I simply removed NAs and worked with what I had
```R
country <- na.omit(country)
```  

## Unique Words

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
![Unique words]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Unique Country.png)
  
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
![Unique words, averaged each year]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Mean Unique Words Country.png)

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
![Unique word to length ratio, averaged every year]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/percent unique country.png)

And lo and behold, when averaging over the yearlly data, there is a clear trend. Not too dramatic (about a .05% decrease every year), but it's still something, and there is more data to be sprunged. 

## Yearly Sentiments
There's a host of sentiment analysis packages out there, most of them hosted in the tidytext library. I decided on afinn, which provides word positivity and negativity on a scale of -5 to 5, where 5 is the most positive of words and -5 is the least.

Using tibble() in the tidytext package and assigned an afinn sentiment to each word, then averaged the sentiment each year to get a general idea of what the songs. 
```R
library(tidytext)

lyrics <- tibble()
for (i in seq_along(country$Year)) {
  clean <- tibble(year = country$Year[i], lyr = as.character(country$Lyrics[i])) %>%
                  unnest_tokens(word, lyr)

  lyrics <- rbind(lyrics, clean)  
}

lyrics <- lyrics %>%
  inner_join(get_sentiments('afinn')) 

sentiment <- as.data.frame(tapply(lyrics$value, lyrics$year, mean))
``` 
![Unique word to length ratio, averaged every year]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Aver sentiment.png)

Because the afinn-scale is positive and negative a perfectly neutral year would have a sentiment of 0. Here we can see that overall Country music is fairly positive, but a bit bleaker in its earlier day. The red line is the overall mean, and between about 1960s to the end of the 90s the average song is happier than usual. Interstingly these are the decades that most people don't have a problem with country music.

## Common Words
Finally I took a look at what would probably be the most telling, commonly used words. I decided that the best way to do this would be to pick out words that people (myself included) associate with *bad* country music. These words would be: truck(s), beer, Jesus/God, America, and gun(s).

Because there are varying song lyrics and lyric lengths each year I decided that the best way to present the data would not be as a total count of the words used, but instead as a percentage showing what percent of the songs each year were made up of these no-no words. 

```R
word_finder <- function(word){
  words <- as.numeric(tapply(country$Lyrics, country$Year, 
                  function(x) sort(decreasing = T, table(clean_string(x))[word])/wordcount(as.character(x))))
  
  words[is.na(words)] <- 0

  return(words)
}

word.count <- data.frame(god = word_finder('god')+word_finder('jesus'), 
                         beer = word_finder('beer'),
                         truck = word_finder('truck')+word_finder('trucks'),
                         US = word_finder('america'),
                         gun = word_finder('gun')+word_finder('guns'))*100


ggplot(word.count) + 
  geom_smooth(aes(x = 1944:2014, y = word.count$god, color = 'God/Jesus'), se = F) +
  geom_smooth(aes(x = 1944:2014, y = word.count$beer, color = 'beer'), se = F) +
  geom_smooth(aes(x = 1944:2014, y = word.count$truck, color = 'truck(s)'), se = F) +
  geom_smooth(aes(x = 1944:2014, y = word.count$US, color = 'America'), se = F) +
  geom_smooth(aes(x = 1944:2014, y = word.count$gun, color = 'gun(s)'), se = F) +
  geom_point(aes(x = 1944:2014, y = word.count$god, color = 'God/Jesus')) +
  geom_point(aes(x = 1944:2014, y = word.count$beer, color = 'beer')) +
  geom_point(aes(x = 1944:2014, y = word.count$truck, color = 'truck(s)')) +
  geom_point(aes(x = 1944:2014, y = word.count$US, color = 'America')) +
  geom_point(aes(x = 1944:2014, y = word.count$gun, color = 'gun(s)')) +
  xlab("Year") + ylab("Percent of Word usage") + ggtitle("Percentage of Words Used")
``` 
![Unique word to length ratio, averaged every year]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Word Use All.png)

Here we can see there there is a sharp uptick of words in our list right around the start of the 90s, particularly in God/Jesus and beer. I decided to look at God/Jesus usage, and interestingly the usage for those words shoot up just after the 1969 and just after 2001, those years being the start of the Vietnam War draft, and the events of 9/11 respectively.

![Unique word to length ratio, averaged every year]({{site.url}}{{site.baseurl}}/Data Science Blog/images/Country Lyrics/Word Use God.png)

## Conclusion
It's difficult to place a timestamp on when music "got bad" or "was good" because of it's sujectivity. Based on what we've seen we can take a guess to when country music changed, however, and because a different beast. The average percentage of unique words per song decreases over time. There's also a huge increase of sentiment in the end of 20th century with a small spike in the first few years of the 21st. And finally words associated with country music, to the point where it's essentially a cliche, have a strong correlation with country wide angst. It also appears that the words take a slight dip near the 80s and into the 90s, and then are used again after 2001 at an even greater rate than before. 

Taking all this into account country music can be divded into three eras:

* 1944-1975 - **Classic Country** Down to earth, a little depressing, very acoustic. Essentially, or literally, folk music often singing about simple everyday things.    
* 1976-9/11/2001 - **Rock Country** Upbeat, electic dance music about little kids kicking the crap out of satan in villion-offs. The country is doing well financially and has less to worry about without a draft or major wars (Cold War not withstanding) and it shows in music.  
* 9/12/2001-Present - **Modern Country** Returning to it's down to earch roots, but this time heavily featuring conservative imagery such as God and guns and America, very likely as a reaction to the events of 9/11.


[back](../../)















