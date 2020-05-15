---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
First, we read in the data and change the date column to the Date class:


```r
activity_data <- read.csv('./activity.csv')
activity_data$date <- as.Date(as.character(activity_data$date), "%Y-%m-%d")
```

## Histogram of the total number of steps taken each day
Now we calcluate the total number of steps taken per day, by using the dplyr
package and grouping the table by the day column and finally summing up the steps per day. This data is then used to display it through a histogram: 


```r
library(dplyr)
activity_per_day <- activity_data %>% group_by(date) %>% summarize(sum(steps))

hist(activity_per_day$`sum(steps)`, main =" Total number of steps taken per day", xlab = "Sum of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
dev.copy(png, "firsthist.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

### What is mean and median total number of steps taken per day?
The mean and median number is calculated through the matching functions and the already grouped data, while ignoring the NA-values:


```r
mean(activity_per_day$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(activity_per_day$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?
To identify the average daily activity pattern and create a time series plot we omit the NA-Values, and group the data by the intervals instead of the date and plot it:


```r
avg_data <- na.omit(activity_data)
avg <- avg_data %>% group_by(interval) %>% summarise(mean(steps))
with(avg, plot(interval, `mean(steps)`, type = "l", main = "Time series plot of average number of steps taken per interval", ylab = "Average steps"))
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

```r
dev.copy(png,"firstTimeSeries.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```
### Which 5-minute interval, on average across all the days contains the maximum number of steps ? 
To answer this question we grab the row, with the max value in the mean(steps)-column of our just calculated values:


```r
avg[avg$`mean(steps)` == max(avg$`mean(steps)`),]
```

```
## # A tibble: 1 x 2
##   interval `mean(steps)`
##      <int>         <dbl>
## 1      835          206.
```

## Imputing missing values

First we have to calculate and report total number of missing values:

```r
sum(is.na(activity_data))
```

```
## [1] 2304
```

To impute missing values, I go by the strategy to add the mean of the specific 5-minute interval, where the value is missing.

This is accomplished by using a for loop, in which i check for NA values, if there is a NA value, it will be replaced by the mean of the specific interval. The transforming will be done to a copy of the original data, to get a new table, as instructed in the task.


```r
library(data.table)
full_activity_data <- copy(activity_data)
for(i in 1:nrow(full_activity_data)){
    if(is.na(full_activity_data[i,1])){
        full_activity_data[i,1] <- avg[(full_activity_data[i,3] == avg$interval),2]
    }
}
```

Now we redo the steps of building a histogram and calculating mean and median, with the new data table.


```r
full_activity_per_day <- full_activity_data %>% group_by(date) %>% summarize(sum(steps))

hist(full_activity_per_day$`sum(steps)`, main =" Total number of steps taken per day", xlab = "Sum of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

```r
dev.copy(png, "secondHist.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

```r
mean(full_activity_per_day$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(full_activity_per_day$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10766.19
```

As we can see there is a huge impact of the midportion of the histogram with steps between 10k to 15k since the y-axis grew from max 25 to 35. On the other side there no major change in the mean or median, since the mean is the same and the median just changed from 10765 to 10766.19.

So the impact mainly lies on the total number of steps taken per day and not changing any mean or median for any intervals. 

## Are there differences in activity patterns between weekdays and weekends?
Now we create two new columns, one with the specific weekday and then a factor variable with the levels "weekday" and "weekend" for a subsequent comparison:

```r
full_activity_data$date <- as.Date(as.character(full_activity_data$date), "%Y-%m-%d")
full_activity_data$weekday <- weekdays(full_activity_data[,2])

for(i in 1:nrow(full_activity_data)){
    if(full_activity_data[i,4] == "Sonntag" |full_activity_data[i,4] == "Samstag" ){
        full_activity_data[i,5]<-"Weekend"}
    else{full_activity_data[i,5]<-"Weekday" }}
```
Using the lattice package two time series plots are created, for comparisom between weekend and weekday activity pattern. 
Using the dplyr package the data is grouped by the factor and the intervals and then creating a mean value for a founded comparison:

```r
library(lattice)
finaltask <- full_activity_data %>% group_by(V5, interval) %>% summarise(mean(steps))
xyplot( `mean(steps)` ~ interval | V5, data = finaltask, type = "l", layout = c(1,2))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

```r
dev.copy(png, "lastTimeSeries.png")
```

```
## png 
##   3
```

```r
dev.off()
```

```
## png 
##   2
```

As we can see there is a different pattern, that througut the day on weekends there is a higher average step count after the first peak which is similar to weekdays at around 800. 