---
title: "PA1_template"
author: "Jed Aureus Gonzales"
date: "Thursday, 14 August, 2014"
---

# Reproducible Research: Peer Assessment 1

This exercise requires you to have the `data.table`, `ggplot2` and `lubridate`
packages installed. If the packages are already installed, run the following:


```r
require (data.table)
require (ggplot2)
require (lubridate)
```

## Loading and preprocessing the data

If `activity.csv` does not exist in your working directory, download it from
[here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip).
We assign it to a variable `activity` instead of `data` so that there wouldn't
be a conflict with parameters that will be used later.

```r
activity.file <- "activity.csv"
if (!file.exists (activity.file)) {
    download.file(url="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip",
                  destfile="repdata-data-activity",
                  method="curl")

    unzip(activity.file)
}
```

Once the data has been unzipped, apply `read.csv()` on it, and typecast
the `date` variable so that it would be easily processed.

```r
activity <- read.csv("activity.csv", stringsAsFactors=FALSE)
activity$date <- as.Date(activity$date)
```

## What is mean total number of steps taken per day?

This question is divided into two parts. First, aggregate the steps against
the date, then plot out the number of steps per day using a histogram:

```r
stepsPerDay <- aggregate(steps ~ date, data = activity, sum, na.rm = TRUE)
hist(stepsPerDay$steps, breaks = 20, main = "Steps per Day", xlab = "Steps")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4.png) 

Next, we calculate the **mean** and the **median** of the steps:

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

This question requires us to use a time series plot of the 5-minute and the
average number of steps taken, averaged across all days. This is done by
aggregating the steps against the interval, the plot out the number of steps
per interval using a time series plot (`type = "l"`):

```r
stepsPerInterval <- aggregate(steps ~ interval, data = activity, mean, na.rm = TRUE)
plot(stepsPerInterval$steps, type = "l", xlab = "Interval", ylab = "Steps")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

With regards to identifying the maximum Step per Interval, we use the
`which.max()` function:

```r
stepsPerInterval[which.max(stepsPerInterval$steps),]
```

```
##     interval steps
## 104      835 206.2
```

## Inputing missing values

As with any data that wasn't collected in ideal conditions, it is almost 
unavoidable to not have `NA`.

First, we calculate the number of `NA`s by using `sum()`:

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

As a means of filling up the missing values, one strategy that could be used
is to supply it with the **mean** number of steps for that time interval. This
strategy is achieved by performing the following:

```r
activityNA <- activity
activityNA$steps[is.na(activityNA$steps)] <- stepsPerInterval$steps
stepsPerDayNA <- aggregate(steps ~ date, data = activityNA, sum, na.rm = TRUE)
hist(stepsPerDayNA$steps, breaks = 20, main = "Steps per Day", xlab = "Steps")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9.png) 

The histogram above looks substantially different than the earlier histogram 
where the `NA` values were just dropped during aggregation. We can also 
examine the difference of the **mean** and **median** of the table with
filled in `NA`s versus the earlier table which dropped the `NA`s.

```r
mean(stepsPerDayNA$steps)
```

```
## [1] 10766
```

```r
median(stepsPerDayNA$steps)
```

```
## [1] 10766
```

The strategy of using the average number of steps per interval to fill the
missing values yielded in a **mean** and **median** not far from the earlier
results. This follows logic where the average would just reinforce itself.

## Are there differences in activity patterns between weekdays and weekends?

First, we need to determine which records are weekdays and weekends. We
can do this by extracting the dates that are weekends and aggregating them
to a separate table from records which fall on weekdays:


```r
ext <- weekdays(activityNA$date) == "Saturday" | weekdays(activityNA$date) == "Sunday"
activityNA$day[ext] <- "weekend"
activityNA$day[!ext] <- "weekday"
activityNA$day <- factor(activityNA$day)
weekend <- aggregate(steps ~ interval, data = activityNA[activityNA$day == "weekend",], mean, na.rm = TRUE)
weekday <- aggregate(steps ~ interval, data = activityNA[activityNA$day == "weekday",], mean, na.rm = TRUE)
```

Next, we visualize them side-by-side to see their visual differences:


```r
par(mfrow = c(2, 1))
plot(weekend$steps, type = "l", main = "Weekend", 
     xlab = "Interval", ylab = "Steps")
plot(weekday$steps, type = "l", main = "Weekday", 
     xlab = "Interval", ylab = "Steps")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12.png) 
