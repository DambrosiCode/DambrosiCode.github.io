## Cursory Data Check
I took my nice chart over to R to take a closer look at the stats, and make some nice graphs and a map. 

First I made quick and dirty plot with base R just to see if there are any outliers I should get rid of.
```R
library(maps)
library(ggplot2)
library(outliers)


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

I opted to remove the first and last row, as they seemed misleading, and move on
```R
stipend.dat.clean <- stipend.dat[-lw.outlier[c(1,6)],]

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
``` 
||lw.ratio.mean|
|-|------------|
Alaska          |   1.4400000
South Dakota    |   1.3940000
Rhode Island    |   1.3032432
Connecticut     |   1.2663265
Missouri        |   1.2417391
New Jersey      |   1.2275000
Illinois        |   1.2210672
North Carolina  |   1.1904514
Arkansas        |   1.1781818
California      |   1.1483179
Michigan        |   1.1429861
New York        |   1.1353474
Pennsylvania    |   1.1103226
Nebraska        |   1.1100000
New Hampshire   |   1.1081250
Minnesota       |   1.1053030
Washington      |   1.1011429
Georgia         |   1.1000000
Texas           |   1.0932258
Arizona         |   1.0903191
Iowa            |   1.0608696
Massachusetts   |   1.0474021
Virginia        |   1.0402778
Tennessee       |   1.0352632
Oregon          |   0.9962264
Indiana         |   0.9923490
Utah            |   0.9876744
West Virginia   |   0.9860000
Vermont         |   0.9850000
Maryland        |   0.9796269
Ohio            |   0.9769032
Delaware        |   0.9714815
Florida         |   0.9671318
New Mexico      |   0.9580000
Kentucky        |   0.9537500
Wisconsin       |   0.9532632
Montana         |   0.9427273
Kansas          |   0.9392593
Idaho           |   0.9183333
Maine           |   0.9050000
Colorado        |   0.8877320
South Carolina  |   0.8604348
Oklahoma        |   0.8600000
Alabama         |   0.8582353
North Dakota    |   0.8522222
Mississippi     |   0.8417647
Louisiana       |  0.8406818
Nevada          |   0.8346667
D.C.            |   0.7974074
Wyoming         |   0.7183333
Hawaii          |   0.6368421

As you can see, Alaska pays very nicely, almost 1.5 times a livable wage, while Hawaii you're payed in good weather and friendly locals. 

```R

``` 
```R

``` 
```R

``` 
```R

``` 
