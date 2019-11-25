A while ago I decided to scrape PhD stipend data off of [this]() website and [find the best states for a living wage](). I also made an interactive map you can check out [here](). I decided to dust the cobwebs off this dataset and play around a bit more. This time I wanted to test a few machine learning algorithms and see if I could predict the stipend based on some predictor variables. 

I am by no means a machine learning expert, but I have a background in statistics and honestly some machine learning libraries are so easy to use now I could teach your grandmother to use it. Grandmother's aside I first had to pick a library to work with, and settled on SKlearn because it's relatively user friendly and well documented. 

You can peak at the data I'm working with [here](). The data itself contains 43 variables, including biking paths present, if there's a bicycling path, and postcode. We're only interested in one thing here, predicting stipend (that is the literal dollar amount you'll get paid to be a student, not the living wage ratio we used previously with this data) so you can imagine some of these data points (like cycling path) won't have much of an effect. Further, since most ML algorithms are data hungry we want don't want a lot of missing data. So we'll use three predictor variables for now: State, University, and Subject. 


```
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np

#We'll be coorelating LW with State, Subject, and University
categories = ['LW Ratio', 'Stipend', 'State', 'University', 'Subject']
stip = pd.read_csv('./stipend_and_locations.csv', encoding = 'latin-1')[categories]
stip = stip.replace(r'^\s*$', np.nan, regex=True) #remove commas from stipend

stip = stip.dropna() #remove missing data
```

One of the big problems with this data is the variability of the Subject names. Unlike State and University variables, Subject doesn't have a naming convention as one university might call a subject Biology and another may call it Biological Studies. To fix this we're goign to try grouping the subjects with similar names under one umbrella term using the library FuzzyWuzzy. This finds the Levenshtein distance between two strings. In other words, we can use this process to find Subjects with roughly the same name that are likely to be related, and give them the same name so the computer recognizes them as the same. I won't post the code here as it's a bit long and a bit of detour, but you can see it in the repository and using it does signifcantly reduce MSE and R^2 we'll get to later. 

Finally we have to decide on how to best model the data. The most obvious choice is using regression to get an estimate. Other options include random forest model, and a neural network. But wait, all of our data is categorical, so how can we use it to predict something continuous? For that we'll use one-hot encoding, wherein each category in a variable is turned into its own variable with a binary True or False value. For instance a table that looked like this:

| University |
|------------------------|
| Stockton University |
| Stony Brook University |
| Seton Hall University |

would look like this when one-hot encoded

| Stockton University | Stony Brook Universty | Seton Hall University |
|---------------------|-----------------------|-----------------------|

This gives the computer something recognizable (numbers) it can read. Alternatively we could have encoded the data as descriptive numbers, where all instances of Stockton University would be replaced by 1, Stony Brook University would be 2, etc. However this becomes problematic when math is performed on the numerical placeholders, for instance if Stockon is 1 and Seton Hall is 3, their average would be 2 which is Stony Brook. But the average of Stockton and Seton Hall isn't Stony Brook because you can't average categorical values such as these. 



And with processed data we're ready to start. 
| 1 | 0 | 0 |
| 0 | 1 | 0 |
| 0 | 0 | 1 |
