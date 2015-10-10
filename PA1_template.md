---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data

```r
## load the dplyr libary - I prefer this for grouping and summarizing
## load ggplot2 for the plots/graphs
library (dplyr)
library (ggplot2)

## this assumes you have downloaded, unzipped, and placed activity.csv into
## your working directory
df_orig <- read.csv("activity.csv")

head(df_orig)
```

```
##   steps      date interval
## 1    NA 10/1/2012        0
## 2    NA 10/1/2012        5
## 3    NA 10/1/2012       10
## 4    NA 10/1/2012       15
## 5    NA 10/1/2012       20
## 6    NA 10/1/2012       25
```

```r
## I had some issues with dates later, so this is ensuring the dates are of the date class
df_orig$date <- as.Date(df_orig$date , format="%m/%d/%Y")

## this dataframe will contain the original without the NAs
df_No_NAs <- na.omit(df_orig)
```

## What is mean total number of steps taken per day?

###Calculate the total number of steps taken per day


```r
## dplyr functions - create the group_by dataframe to use in the summarize function
byDates <- group_by(df_No_NAs, date)

## summarize to get the daily totals, 

dailyTotals <- summarize(byDates, sum=sum(steps))  

print(dailyTotals)
```

```
## Source: local data frame [53 x 2]
## 
##          date   sum
## 1  2012-10-02   126
## 2  2012-10-03 11352
## 3  2012-10-04 12116
## 4  2012-10-05 13294
## 5  2012-10-06 15420
## 6  2012-10-07 11015
## 7  2012-10-09 12811
## 8  2012-10-10  9900
## 9  2012-10-11 10304
## 10 2012-10-12 17382
## ..        ...   ...
```

###Make a histogram of the total number of steps taken each day


```r
png("figure/plot 1 - dailyTotals.png")
p <- ggplot(data = dailyTotals, aes(x=date, y=sum)) + geom_bar(stat="identity")
print(p)
dev.off()
```

```
## png 
##   2
```

```r
print(p)
```

###Calculate and report the mean and median of the total number of steps taken per day


```r
## Summarize to get the mean and median of the daily totals
dailyMeanMedian <- summarize(dailyTotals, mean=mean(sum), median=median(sum)) 
print(dailyMeanMedian)
```

```
## Source: local data frame [1 x 2]
## 
##       mean median
## 1 10766.19  10765
```

## What is the average daily activity pattern?

###Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
byInterval <- group_by(df_No_NAs, interval)
## summarize by the interval to get the means for each interval
intervalMeans <- summarize(byInterval, mean=mean(steps)) 
##print(intervalMeans)

## create the plot of interval means, one to a file, one to the document
png("figure/plot 2 - intervalMeans.png")
q <- ggplot( data = intervalMeans, aes( interval, mean )) + geom_line() 
print(q)
dev.off()
```

```
## png 
##   2
```

```r
##q <- ggplot( data = intervalMeans, aes( interval, mean )) + geom_line() 
print(q)
```

###Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
maxInterval <- filter(intervalMeans, mean == max(mean))
print(maxInterval)
```

```
## Source: local data frame [1 x 2]
## 
##   interval     mean
## 1      835 206.1698
```

## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
