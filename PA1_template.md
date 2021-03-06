---
title: "Reproducible Research: Peer Assessment 1"
output: 
    html_document: 
       keep_md: true
---



Overview
===
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are: steps date, and interval at which measurement was taken. The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

Reading in & Process Data 
===
After downloading the files: setting my wd, unzipping, and reading in file. 

```r
setwd("D:/R/reproducible research/Repdata")
unzip("./activity.zip", exdir = "D:/R/reproducible research/Repdata")
act <- read.csv("activity.csv", header = TRUE)
```

What is mean total number of steps taken per day?
===
visualize the structure of the file & preparing for data manipulation & graphing

```r
str(act)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(act)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
#for manipulating data
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
#for graphing
library(ggplot2)
```

1. Calculate the total number of steps taken per day.


```r
totstep <- act %>%
        group_by(date) %>%
        summarize(sum = sum(steps, na.rm = TRUE))
View(totstep) #for viewing the results
```

2. Histogram of total number of steps taken each day

```r
hist(totstep$sum, xlab="steps", main = "Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

3. Mean and median number of steps taken each day

```r
#The mean of total steps taken each day
Mean <- mean(totstep$sum)
#The median of total steps taken each day
Median <- median(totstep$sum)
```

What is the average daily activity pattern
===
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).

```r
Intervalstep <- act %>%
        group_by(interval) %>%
        summarize(avgstep = mean(steps, na.rm = TRUE))
plot(Intervalstep$interval, Intervalstep$avgstep, main = "average daily activity pattern", xlab = "Interval", ylab = "Mean steps", type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->
2. Which 5-min interval, on avg across all the day in the dataset, contains the maximum number of steps.

```r
#It is asking for max mean values, so we use intervalstep 
MaxI <- Intervalstep$interval[which.max(Intervalstep$avgstep)]
MaxI
```

```
## [1] 835
```

```r
#The interval is the one at 835.
```
Inputting Missing values
===
1. Calculate total # of rows with NA

```r
sum(is.na(act))
```

```
## [1] 2304
```

```r
#The result yields 2304 rows with NA values. 
```
2. Strategy for filling in missing values is to replace them with the mean of that 5-min interval. 

3.This strategy will replace the NA whenever the interval matches. 

```r
newact <- act
sum(is.na(newact)) #2304
```

```
## [1] 2304
```

```r
head(newact)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```

```r
for (i in 1:nrow(act)){
        if (is.na(act$steps)[i] == TRUE) {newact$steps[i] <- Intervalstep$avgstep[Intervalstep$interval == newact$interval[i]]
        }
}
sum(is.na(newact)) #0 NA
```

```
## [1] 0
```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
totstepnew <- newact %>%
        group_by(date) %>%
        summarize(newsum = sum(steps, na.rm = TRUE))
hist(totstepnew$newsum, xlab = "steps", main = "new dataset's histogram")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
--- Reporting the mean and median
Both the mean and median are different.

```r
mean(totstepnew$newsum) #New mean
```

```
## [1] 10766.19
```

```r
median(totstepnew$newsum) #New median 
```

```
## [1] 10766.19
```

```r
Mean #Old mean
```

```
## [1] 9354.23
```

```r
Median #Old median
```

```
## [1] 10395
```
Are there differences in actiity patterns between weekdays and weekneds? 
===
1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
newact$date <- as.Date(newact$date)
newact$weekday <- weekdays(newact$date)
newact$weekend <- ifelse(newact$weekday %in% "Sunday" | newact$weekday %in%"Saturday", "Weekend", "Weekday")
newact$weekend <- as.factor(newact$weekend)
```
2.Make a panel plot containing a time series plot  of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
#General thought process is to use aggregate and then ggplot. 
Datasetfinal <- aggregate(newact$steps, by=list(newact$weekend,newact$interval ), mean)
names(Datasetfinal)
```

```
## [1] "Group.1" "Group.2" "x"
```

```r
# Renaming columns
names(Datasetfinal) <- c("weekend", "interval", "steps")
# ploting 
library(ggplot2)
ggplot(Datasetfinal, aes(interval, steps, weekend)) +
        geom_line() +
        facet_grid(.~ weekend) +
        ylab("mean steps") +
        xlab("intervals") +
        ggtitle("Mean steps vs Intervals on weekdays and weekends")
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->




