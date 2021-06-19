---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


```r
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
library(ggplot2)
```

## Loading and preprocessing the data

1. Load the data (i.e. read.csv()）


```r
# read csv file
health <- read.csv(unzip("activity.zip"))
```

2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
# transform date 
health$date <- as.Date(health$date)
```

## What is mean total number of steps taken per day?

2. Histogram of the total number of steps taken each day


```r
health1 <- health %>% group_by(date) %>% summarise(total_steps=sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
hist(health1$total_steps, breaks = seq(0,25000,2500),
        main="Histgram of the Total number of steps taken each day",
        xlab="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

3. Mean and median number of steps taken each day


```r
mean(health1$total_steps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(health1$total_steps, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

4. Time series plot of the average number of steps taken


```r
health2 <- health %>% 
  filter(!is.na(steps)) %>% 
  group_by(interval) %>% 
  summarise(mean_steps=mean(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
plot(health2$interval, health2$mean_steps, type = "l",
        main="Time series plot of the average number of steps taken",
        xlab="Interval",
        ylab="Average of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

5. The 5-minute interval that, on average, contains the maximum number of steps


```r
max_steps <- max(health2$mean_steps)
health2[health2$mean_steps == max_steps,]
```

```
## # A tibble: 1 x 2
##   interval mean_steps
##      <int>      <dbl>
## 1      835       206.
```

## Imputing missing values

6. Calculate and report the total number of missing values in the dataset


```r
health3 <- health %>% filter(is.na(health$steps)) %>% summarise(missing_value=n())

health3
```

```
##   missing_value
## 1          2304
```

6A.Devise a strategy for filling in all of the missing values in the dataset

Fill NA values using the mean of steps for each interval in that day


7. Histogram of the total number of steps taken each day after missing values are imputed


```r
health4 <- health %>% 
  filter(!is.na(steps)) %>% 
  group_by(interval) %>% 
  summarise(mean_steps=mean(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
noNAhealth4 <- merge(health4, health)
NAhealth <- is.na(noNAhealth4$steps)
noNAhealth4$steps[NAhealth] <- noNAhealth4$mean_steps[NAhealth]

health5 <- noNAhealth4 %>%
  group_by(date) %>%
  summarize(total_steps=sum(steps))
```

```
## `summarise()` ungrouping output (override with `.groups` argument)
```

```r
hist(health5$total_steps, breaks = seq(0,25000,2500),
        main="Histgram of the Total number of steps taken each day",
        xlab="Total number of steps taken each day")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

## Are there differences in activity patterns between weekdays and weekends?

8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends


```r
health6 <- health
health6$weekday <- weekdays(health6$date)
health6$daytype <- ifelse(health6$weekday=="土曜日" | health6$weekday=="日曜日", "weekend","weekday")

head(health6)
```

```
##   steps       date interval weekday daytype
## 1    NA 2012-10-01        0  月曜日 weekday
## 2    NA 2012-10-01        5  月曜日 weekday
## 3    NA 2012-10-01       10  月曜日 weekday
## 4    NA 2012-10-01       15  月曜日 weekday
## 5    NA 2012-10-01       20  月曜日 weekday
## 6    NA 2012-10-01       25  月曜日 weekday
```

```r
health7 <- health6 %>% 
  filter(!is.na(health6$steps)) %>% 
  group_by(interval, daytype) %>% 
  summarise(mean_steps=mean(steps))
```

```
## `summarise()` regrouping output by 'interval' (override with `.groups` argument)
```

```r
g <- ggplot(data=health7, aes(x=interval, y=mean_steps)) + 
  geom_line()+
  facet_grid(daytype~.) +
  ggtitle("Average number of steps taken per 5-minute interval") +
  xlab("Interval") +
  ylab("Average of steps") 
g
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

9. All of the R code needed to reproduce the results (numbers, plots, etc.) in the report


