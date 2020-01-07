---
title: "PhD Stipends Part I: The Code"
date: 2020-01-07
tags: [Statistics, Interactive, Map, Stipend Stats]
category: "Data Science"
excerpt: "Statistics"
---
## Cursory Data Check
I took my nice chart over to R to take a closer look at the stats, and make some nice graphs and a map.

First I made quick and dirty plot with base R just to see if there are any outliers I should get rid of.
```R
library(maps)
library(ggplot2)
library(outliers)
library(dplyr)


stipend.dat <- read.csv('C:/Users/mattd/Desktop/Projects/Python Projects/Stipe And Locations.txt', sep='\t')


#FIND OUTLIERS
#let's see what a quick model looks like
stipend.dat <- stipend.dat[order(stipend.dat$LW.Ratio), ]
plot(stipend.dat$LW.Ratio, ylab = 'LW Ratio', main = 'LW Ratio per University')
```

It definitely looks like there are. Using the glm() function to get a generalized linear model, I used Cook's Distance and the Outliers library to figure out which points were the most outlying.
```R
#get a generalized linear model
lw.glm <- glm(1:length(stipend.dat$LW.Ratio)~stipend.dat$LW.Ratio)

#get outliers
lw.cooks <- cooks.distance(lw.glm)
lw.outlier <- c(which(lw.cooks >= mean(lw.cooks)*4))

stipend.dat[lw.outlier,c('Subject', 'LW.Ratio', 'Stipend')]
```
| Subject | LW.Ratio | Stipend |
|---------|----------|---------|
Information Technology | -34.01 | -900000
statistics | 12.78 | 310000
Biological sciences | 13.14 | 340000
SEMTE | 13.45 | 298500
Strategy | 13.50 | 300000
Finance | 40.97 | 994000

I opted to remove last row, as it seemed misleading at almost a stipend of $900,000. As well as stipends that payed no money or less IE: the students weren't getting paid but instead had to pay to go.
```R
#remove outliers
stipend.dat.clean <- stipend.dat[-lw.outlier[6],]

#remove <0
stipend.dat.clean <- stipend.dat.clean[which(stipend.dat.clean$Stipend > 0),]
```

## Stimulating Stipend Stats
It was time to get State-wide data, however, there were a few weirdo state names that (last time I checked) aren't states. There were only 4 of them, so it wasn't too bad to remove these rows.
```R
#GET BASIC STATS
lw.ratio.mean <- sort(tapply(stipend.dat.clean$LW.Ratio, stipend.dat.clean$State, mean), decreasing = T)

#remove weirdos
stipend.dat.clean <- subset(stipend.dat.clean, State != 'CAT' & State != 'LAZ' & State != '' & State != 'Auvergne-RhÃ´ne-Alpes')

#average by state
lw.ratio.mean <- sort(tapply(stipend.dat.clean$LW.Ratio, stipend.dat.clean$State, mean), decreasing = T)

colMeans(lw.ratio.mean)
```
||lw.ratio.mean|
|-|------------|
Alaska         | 1.4400000
South Dakota    | 1.3940000
Illinois        | 1.3319048
Rhode Island    | 1.3283333
Arkansas       |  1.2890000
Connecticut     | 1.2815464
Missouri       |  1.2804478
North Carolina  | 1.2567647
New Jersey      | 1.2452381
Michigan        | 1.1855797
California      | 1.1825397
New York        | 1.1683489
Texas           | 1.1572549
Washington      | 1.1482090
Georgia        |  1.1480392
Pennsylvania   |  1.1467286
Massachusetts   | 1.1223485
Iowa            | 1.1211628
Oregon          | 1.1210870
Minnesota       | 1.1175385
Tennessee       | 1.1158621
Arizona         | 1.1141304
Nebraska        | 1.1100000
New Hampshire   | 1.1081250
Maine           | 1.0700000
Virginia        | 1.0680952
Indiana         | 1.0577857
Ohio            | 1.0514894
Montana         | 1.0310000
Colorado        | 1.0237647
Utah            | 1.0217073
Wisconsin       | 1.0150000
New Mexico      | 0.9985714
Oklahoma        | 0.9951852
Kentucky        | 0.9875000
Maryland        | 0.9860902
West Virginia   | 0.9860000
Vermont         | 0.9850000
Florida         | 0.9838889
Kansas          | 0.9738462
Delaware        | 0.9714815
Wyoming         | 0.9650000
South Carolina  | 0.9530000
Mississippi     | 0.9392857
Idaho           | 0.9183333
Louisiana       | 0.9130769
North Dakota    | 0.8962500
Alabama         | 0.8956250
Nevada          | 0.8892857
D.C.            | 0.8111321
Hawaii          | 0.6694444

As you can see, Alaska pays very nicely, almost 1.5 times a livable wage, while Hawaii you're payed in good weather and friendly locals.

```R
mean(stipend.dat.clean$LW.Ratio)
```
      [1] 1.131833
```R
mean(stipend.dat.clean$Stipend)
```
      [1] 26375.38

At an average country-wide mean of 1.13, the average student has a only 13% of their income that won't go to necessities such as food, room and board, and transportation before taxes. An average stipend of $26375.38, this comes to $3428.80 per year for eating out, movies, and savings after graduation. Not to mention any loans the student might have to pay off.  

Taking a look at average livable wage per-subjects, there are a few subjects that don't have too much data on them so it doesn't really help to average. By only looking at subjects that have 5 or more datapoints, the results become much more clear cut.
```R
#remove data with one point
unrow <- table(stipend.dat.clean$Subject)
no.uni <- stipend.dat.clean[stipend.dat.clean$Subject %in% names(unrow)[unrow > 5],]

#average LW ratio per subject
sort(tapply(no.uni$LW.Ratio, tolower(no.uni$Subject), mean), decreasing = T)[c(1:5,104:109)] %>%
  data.frame(.)
```
|Subject|LW Ratio|
|-------|--------|
clinical psychology   | 1.4840000
chemical engineering  | 1.3044828
neuroscience          | 1.2952830
biomedical engineering | 1.2834286
aerospace engineering | 1.2675000
english               | 0.8961594
communication         | 0.8791667
social work           | 0.8787500
english               | 0.8583333
geography             | 0.8168421
spanish               | 0.7450000

No surprise, marketable subjects, like engineering can afford to pay the bset stipends.

## Making Maps to Marvel
Finally, I wanted to make a map, so I knew which states to avoid.

First step was merging the averaged data to the usmap library by state name
```R
library(usmap)
library(ggiraph)
states <- us_map(region = 'states')

#merge the two data sets
lw.ratio.mean['State'] <- tolower(rownames(lw.ratio.mean))
states.lw = merge(x = states, y = lw.ratio.mean, by.x = 'full', by.y = 'State', all.x = T)
states.lw$lw.ratio.mean = round(states.lw$lw.ratio.mean,digits = 3)
```
Simple enough. Then it was just a matter of plotting with ggplot2 and making it interactive with ggiraph
```R
oc <- "alert(this.getAttribute(\"data-id\"))"
g <- ggplot(data = states) +
  geom_polygon_interactive(aes(x = long, y = lat, fill = states.lw$lw.ratio.mean, group = group,
                               tooltip=states.lw$lw.ratio.mean,
                               data_id = states.lw$full, onclick = oc), color = "white") +
  coord_fixed(1) + labs(fill = 'Living-Wage \n Ratio') + ggtitle('Average Stipend per State')

ggiraph(code=print(g))
```
Now the map will give you an overview of the LW ratio and state name when you click on a state. 

{% include stpend_map.html %}
