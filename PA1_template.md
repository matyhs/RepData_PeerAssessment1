---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


```r
library(knitr)
opts_chunk$set(echo = TRUE, results = "asis")
```

## Loading and preprocessing the data


```r
if(!file.exists("activity.csv")){
    unzip("activity.zip")
}

data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day


```r
groupdata<- aggregate(data$steps, list(data$date), sum, na.rm=TRUE)
kable(groupdata, 
      col.names = c("Date", "Total Steps"), align = 'l', 
      caption = "Total Steps Per Day")
```



Table: Total Steps Per Day

Date         Total Steps 
-----------  ------------
2012-10-01   0           
2012-10-02   126         
2012-10-03   11352       
2012-10-04   12116       
2012-10-05   13294       
2012-10-06   15420       
2012-10-07   11015       
2012-10-08   0           
2012-10-09   12811       
2012-10-10   9900        
2012-10-11   10304       
2012-10-12   17382       
2012-10-13   12426       
2012-10-14   15098       
2012-10-15   10139       
2012-10-16   15084       
2012-10-17   13452       
2012-10-18   10056       
2012-10-19   11829       
2012-10-20   10395       
2012-10-21   8821        
2012-10-22   13460       
2012-10-23   8918        
2012-10-24   8355        
2012-10-25   2492        
2012-10-26   6778        
2012-10-27   10119       
2012-10-28   11458       
2012-10-29   5018        
2012-10-30   9819        
2012-10-31   15414       
2012-11-01   0           
2012-11-02   10600       
2012-11-03   10571       
2012-11-04   0           
2012-11-05   10439       
2012-11-06   8334        
2012-11-07   12883       
2012-11-08   3219        
2012-11-09   0           
2012-11-10   0           
2012-11-11   12608       
2012-11-12   10765       
2012-11-13   7336        
2012-11-14   0           
2012-11-15   41          
2012-11-16   5441        
2012-11-17   14339       
2012-11-18   15110       
2012-11-19   8841        
2012-11-20   4472        
2012-11-21   12787       
2012-11-22   20427       
2012-11-23   21194       
2012-11-24   14478       
2012-11-25   11834       
2012-11-26   11162       
2012-11-27   13646       
2012-11-28   10183       
2012-11-29   7047        
2012-11-30   0           

2. Make a histogram of the total number of steps taken each day


```r
hist(groupdata$x, xlab="Total Steps", main="Histogram of Total Steps")
```

![](PA1_template_files/figure-html/histogramsteps-1.png)<!-- -->

3.Calculate and report the mean and median of the total number of steps taken per day


```r
print(paste("Mean of the total number of steps = ", mean(groupdata$x)))
```

[1] "Mean of the total number of steps =  9354.22950819672"

```r
print(paste("Median of the total number of steps = ", median(groupdata$x)))
```

[1] "Median of the total number of steps =  10395"

## What is the average daily activity pattern?

1. Make a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days


```r
groupdata <- aggregate(data$steps, list(data$interval), mean, na.rm=TRUE)
with(groupdata, plot(Group.1, x, type = "l", xlab = "Interval", 
                     ylab = "Average Steps", 
                     main = "Average Steps Per Interval"))
```

![](PA1_template_files/figure-html/stepsperinterval-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
print(paste("Interval ", groupdata[which.max(groupdata$x),1], " is the maximum interval with average steps of ", groupdata[which.max(groupdata$x),2]))
```

[1] "Interval  835  is the maximum interval with average steps of  206.169811320755"

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset


```r
missing <- is.na(data$steps)
print(paste("Number of missing values = ", sum(missing)))
```

[1] "Number of missing values =  2304"

2. Devise a strategy for filling in all of the missing values in the dataset.

Strategy used is to fill the missing values with the mean for that 5-minute interval

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
imputdata <- data
names(groupdata) <- c("interval", "steps")
imputdata <- merge(imputdata, groupdata, by = "interval")
imputdata[is.na(imputdata$steps.x), ]$steps.x <- 
    imputdata[is.na(imputdata$steps.x),]$steps.y
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

Based on the graphs below, there is a significant difference from using the original data versus the data with imputted valeus, causing an increase with the total number of steps.


```r
groupdata<- aggregate(imputdata$steps.x, list(imputdata$date), sum, na.rm=TRUE)
hist(groupdata$x, xlab="Total Steps", main="Histogram of Total Steps")
```

![](PA1_template_files/figure-html/meanmedianimputsteps-1.png)<!-- -->

```r
print(paste("Mean of the total number of steps = ", mean(groupdata$x)))
```

[1] "Mean of the total number of steps =  10766.1886792453"

```r
print(paste("Median of the total number of steps = ", median(groupdata$x)))
```

[1] "Median of the total number of steps =  10766.1886792453"

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
imputdata$factor <- ifelse(weekdays(as.Date(imputdata$date))%in%c("Saturday", "Sunday"), "Weekend", "Weekday")
```

2. Make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days


```r
library(ggplot2)
groupdata <- aggregate(data$steps, list(data$interval), mean, na.rm=TRUE)
g <- ggplot(imputdata, aes(interval, steps.x)) + labs(x = "Interval", y = "Average Imputted Steps", title = "Average Imputted Steps Per Interval")
g  + stat_summary(fun.y = mean, geom="line", size=1) + facet_grid(factor~.) 
```

![](PA1_template_files/figure-html/dayofweekplot-1.png)<!-- -->
