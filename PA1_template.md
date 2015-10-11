---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

# Reproducible Research: Peer Assessment 1
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

## these dataframes will contain the original with and without the NAs - to be used in subsequent steps
df_No_NAs <- na.omit(df_orig)
## go back to the original dataframe and create a logical vector, then use it to subset the original
## to have a new dataframe with only the NAs
df_sub <- is.na(df_orig)
df_NAs = df_orig[df_sub,]
```

## What is mean total number of steps taken per day?

### Calculate the total number of steps taken per day


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

### Make a histogram of the total number of steps taken each day


```r
p <- ggplot(data = dailyTotals, aes(x=date, y=sum)) + geom_bar(stat="identity")
print(p)
```

![plot of chunk plot1 - histogram - dailyTotals](figure/plot1 - histogram - dailyTotals-1.png) 

### Calculate and report the mean and median of the total number of steps taken per day


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

### Make a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
byInterval <- group_by(df_No_NAs, interval)
## summarize by the interval to get the means for each interval
intervalMeans <- summarize(byInterval, mean=mean(steps)) 
##print(intervalMeans)

## create the plot of interval means, one to a file, one to the document
q <- ggplot( data = intervalMeans, aes( interval, mean )) + geom_line() 
print(q)
```

![plot of chunk plot2 - time series - intervalMeans](figure/plot2 - time series - intervalMeans-1.png) 

### Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

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

### Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
## How many NAs are there?
number_of_NAs <- nrow(df_NAs)
print(number_of_NAs)
```

```
## [1] 2304
```

### The strategy for filling in the missing values in the dataset:
* Use the mean for the missing interval 
* There are two dataframes - with and without NAs
* There is also one with the intervalMeans
* Merge NAs with intervalMeans (by interval)
* Append (rbind) this merged df (df_nas_reorder) with the df_no_nas
* Show the head() rows of the new resulting df - df_all - inital rows were NA at the beginning

```r
## the steps here are NAs - so drop this column - we'll be merging to it later to 
## replace with the interval means
df_NAs$steps <- NULL

## this will change the "mean" column in te intervalMeans df to steps to be 
## consistent with this question.
names(intervalMeans)[2] <- c("steps")

## merge the NAs df with the intervalMeans by the interval colulmn.  This will "Fill in" the 
## missing (NAs) data.
df_NAs2 <- merge(df_NAs, intervalMeans, "interval")
## reorder the columns to match that of the df_orig
df_NAs2_reorder <- df_NAs2[c(3,2,1)]
```

### Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
## append the two data frames together again, sort (arrange) it so the head() is 
## in the same order as the originL
## the sort made it easier to see the results of the filling in the NAs data.
df_all <- rbind(df_No_NAs, df_NAs2_reorder)
df_all <- arrange(df_all, date, interval)

head(df_all)
```

```
##       steps       date interval
## 1 1.7169811 2012-10-01        0
## 2 0.3396226 2012-10-01        5
## 3 0.1320755 2012-10-01       10
## 4 0.1509434 2012-10-01       15
## 5 0.0754717 2012-10-01       20
## 6 2.0943396 2012-10-01       25
```

### Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. 

```r
# using df_all, group and summarize again, to get the Daily Totals, and the DAily Mean 
## and Median of the daily totals
byDates <- group_by(df_all, date)
byInterval <- group_by(df_all, interval)

dailyTotals_all <- summarize(byDates, sum=sum(steps))  
dailyMeanMedian_all <- summarize(dailyTotals_all, mean=mean(sum), median=median(sum))  

p <- ggplot(data = dailyTotals_all, aes(x=date, y=sum))+geom_bar(stat="identity")
print(p)
```

![plot of chunk plot3 - histogram - after imputing missing values](figure/plot3 - histogram - after imputing missing values-1.png) 
### Do these values differ from the estimates from the first part of the assignment?
### What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
## after imputing the missing values
print(dailyMeanMedian_all)
```

```
## Source: local data frame [1 x 2]
## 
##       mean   median
## 1 10766.19 10766.19
```

```r
## print the original mean and median down here again, so that it will be easier to compare
print(dailyMeanMedian)
```

```
## Source: local data frame [1 x 2]
## 
##       mean median
## 1 10766.19  10765
```
### No, there is no impact of imputing the missing data.  Since the mean and median were close to each other, and the strategy to fill in missing values was to use the mean of the missing interval, the new df_all also has the same mean and a very similar median.

=============================================================================
## Are there differences in activity patterns between weekdays and weekends?

### Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
# After much experimenting with the weekdays function, I found this rather elegant formula for 
# creating groups for Weekend vs WeekDay via a web search.
# Then store the result as a new column in the data frame
df_all$typeWeekDay <- ifelse (weekdays(df_all$date) %in%  c("Saturday", "Sunday"),'Weekend','Weekday')
 
# Group and Summarize to get the daily means by interval and type of weekday
byInterval <- group_by(df_all, interval, typeWeekDay)
dailyMeans_all <- summarize(byInterval,mean=mean(steps))  
```

### Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
# create the weekday vs. weekend plot
q <- ggplot( data = dailyMeans_all, aes( interval, mean )) + geom_line()  + facet_grid(typeWeekDay~.) +   
     labs(title="Mean steps per interval, panels by Weekday vs. Weekend")  + facet_wrap(~typeWeekDay,nrow=2)
print(q)
```

![plot of chunk plot4 - time series panels - by weekday vs weekend](figure/plot4 - time series panels - by weekday vs weekend-1.png) 

