---
layout: default
---

## Student Stipend Stats in States 
Nearing the end of my career in academia I have a few choices to make, namely what I want to do when I graduate 
The idea of pursuing a PhD and becoming a professional student is attractive, but not without its pitfalls.
One of my biggest concerns is living expenses. little more than 20 grand a year hardly seems livable,
however, my cohorts seem to all make it work so I deicded to take a closer look and see 
what average stipend looked like based on the living wage in each state. 

### Lifting Lists of Living 
First order of business is getting the data. I looked around a bit and there's shockingly little
data all in one place that gives me the University, Location, Department, and Stipend. Luckily 
[phdstipends.com](http://www.phdstipends.com/results) gives me a University, Department and Stipend as well as that stipend's 
living wage ratio - which saves me a little bit of time since I won't have to calculate it myself. 
It's also updated regularly by PhD students, so it remains fairly up-to-date. 

First step was scraping the website. It was a bit trickier since the actual table wasn't
static I had to use BeautifulSoup as well as Selenium for scraping
```python
from bs4 import BeautifulSoup
import requests
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

url="http://www.phdstipends.com/results"

#get table to open
driver=webdriver.Firefox()
driver.get(url)
```
To get scrape each page I had to automate a window to scrape and then click to the next page. Here
is where Selenium's webdriver came in handy
```python
col=[] #collection of data
page=0 #current page

#this function will parse through the website and collect data on each page
def page_parser():
    #scrape data
    stipend_soup=BeautifulSoup(driver.page_source, 'lxml')
    level1=stipend_soup.find('table',{'id':'mytable'})
    level2=level1.find('tbody')
    global page
    
    #Every time a page is fully scraped, click to the next one
    i = 0
    for stats in level2.findAll('td'):
        col.append(stats.get_text())
        i+=1
        #run for 350 items in table in 128 pages
        if i==350 and page<=128:
            page+=1
            try:
                #click to the 'next page' button and then run the function again
                driver.find_element_by_xpath('//*[@id="mytable_next"]/a').click()
                page_parser()
            except:
                pass
        else:
            pass
    
page_parser()
```
After a minute or two all the data is collected in col variable, but that's not too easy to
work with so I used Pandas and Nummpy to make the data cleaner in the following steps.

```python
import pandas as pd
import numpy as np

#Make seven titled columns
stipend_stats=pd.DataFrame(np.reshape(col,(-1,7)))
stipend_stats.columns=["University",
                       "Subject",
                       "Stipend",
                       "LW Ratio",
                       "Academic Year",
                       "Program Year",
                       "Comments"]
                       
#Save as a .csv so I don't have to worry about re-scraping the data later
stipend_stats.to_csv("stipend_stats.txt",sep='\t')
stipend_stats = pd.read_csv('Stipend_Stats.txt', sep='\t')
```
Since I'm interested in the Living Wage Ratio, I decided to drop the 'Comments' column, and 
then clean every row that had missing data. I wanted to not only clean the data, but make it
a bit smaller since the next step was going to take some time. 
```python
stipend_stats = stipend_stats.iloc[:,:-1].dropna().reset_index(drop=True)
```
This dropped the rows from about 6000 to 5000, but a substantial amount of data remained.

For the next part I needed to get location data, which unfourtunately, is not included on the 
website. To fix this I needed an API that dealt with maps, and I didn't feel like paying one. 
The Nominatin API allows for searches through OpenStreeMaps, a free regularly updated map-database.

The 'geopy.geocoders' library provides this search function, and is relatively easy to use,
requiring only a string that vaguely matches a location. However, I noticed it did work
better when there were no parathenses , so to fix this I cleaned the University name in each row.
```python
import re 
  
#remove anything in parenthesis
def Clean_names(name): 
    if re.search(r'\([^)]*\)', str(name)): 
        pos = re.search(r'\([^)]*\)', str(name)).start() 
        return name[:pos] 
  
    else: 
        return name 

stipend_stats['University'] = stipend_stats['University'].apply(Clean_names) 
stipend_stats

#Clean the Stipend column of all special characters
stipend_stats['Stipend'] = stipend_stats['Stipend'].str.replace(',', '')
stipend_stats['Stipend'] = stipend_stats['Stipend'].str.replace('$', '')
stipend_stats['Stipend'] = pd.to_numeric(stipend_stats['Stipend'])

```

Finally I could systematically search for location, location, location. The one caveat was that
Nomanatin will only allow a max of one search query per second, so the function had to be artificially
slowed down. I could have downloaded OpenStreetMap data files from the web and searched through
them as quickly as I'd like, however, the files were large and my storage is low and the time it'd 
take to download the files would be about the time it'd take to run through with the API. At the end
of the day, this seemed like the best option.
```python
from geopy.geocoders import Nominatim
from tqdm import tqdm
import time

geolocator = Nominatim(user_agent="Stipend")
stipend_stats["Location"]='' #New location column

#get location
def get_locs(i):
    try:
        location = geolocator.geocode(stipend_stats["University"][i], addressdetails=True)
        loc = location.raw
        return(loc['address'])       
    except:
        print("ERROR") #Print ERROR everytime a location can't be found
        return('NA')

    
for i in tqdm(range(len(stipend_stats))):
    stipend_stats["Location"][i] = get_locs(i)
    stipend_stats.to_csv('Stipend.txt', sep='\t')
    time.sleep(.5) #gotta go slow
```

Some of the locations couldn't be found, so I performed a cursory checkup to see how much data
was lost. 
```python
print(len(stipend_stats)-len(stipend_stats.dropna()))/len(stipend_stats)) #only about 15% was lost

#remove the rows with missing locations
stipend_stats = stipend_stats.dropna().reset_index(drop=True) 

#Save to CSV again
stipend_stats = pd.read_csv('Stipend.txt', sep='\t')
```

Since the Location was saved as Dictionary in the Location column I wanted seperate
columns for each Key. 
```python
from ast import literal_eval
#when reading from CSV convert string to dictionary
stipend_stats['Location'] = stipend_stats['Location'].apply(lambda x: literal_eval(x))

#Dictionary keys to columns
loc = stipend_stats['Location'].apply(pd.Series)
#Capitalize first letter in column names
loc.columns = map(str.capitalize, loc.columns)

#merge location and stipend_stats dataframes
stipend_loc = pd.concat([loc, stipend_stats], axis = 1)
```

Finally, one last datacure to clean things up a bit. There were two University columns that had to go now, 
and the Location column was no longer needed.
```python
#remove second University column
stipedn_loc = stipend_loc.iloc[:,~stipend_loc.columns.duplicated()]
#remove Location column
stipend_loc = stipend_loc.drop(['Location', 'Unnamed: 0', 'Unnamed: 0.1'], axis =1 )

#Save to CSV one last time
stipend_loc.to_csv('Stipe And Locations.txt', sep='\t')
```

And thus a beautiful new csv was born. If you'd like to take a look and play with the data 
you can [here](). And some stats and the map I made with the data can be found in part 2.
