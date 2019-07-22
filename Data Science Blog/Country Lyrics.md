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
   
  #### Something's Missing
  
  The first big hurtle I faced was the fact that more than a few songs were missing from the data. I suspected this was because Genius likely doesn't have all the lyrics for every song, and it probably would be the older songs that were less likely to be in Genius's database. 
   
  
[back](../../)















