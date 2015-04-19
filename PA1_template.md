# Reproducible Research: Peer Assessment 1

## Loading and preprocessing the data
1. Set the working directory to the forked project repo. 
2. Unzip the archive containing the activity data
3. Load the activity data csv file into R

```r
setwd("/Users/Simon/datasciencecoursera/RepData_PeerAssessment1")
unzip("activity.zip")
activity<-read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
To calculate the total number of steps taken per day, aggregate the recorded
measurements from all of the 5 minute intervals for each day. 

```r
dailysteps<-aggregate(steps~date, activity, sum)
```

The following histogram organizes the total steps taken each day into 
value bins:

```r
hist(dailysteps$steps, col="red", xlab="Total Daily Steps", 
     ylab="Number of Days", main="Total Number of Steps Taken Per Day")
```

![](./PA1_template_files/figure-html/histogram-1.png) 

The **mean** number of steps taken per day is:

```r
mean(dailysteps$steps)
```

```
## [1] 10766.19
```
The **median** number of steps taken per day is:

```r
median(dailysteps$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

To show how activity varies throughout the day calculate the daily mean 
of recorded steps for each of the 5 minute data collection intervals.

```r
activitypattern<-aggregate(steps~interval, activity, mean)
```

Using these averaged step counts, a time series plot shows activity levels
throughout the data.

```r
with(activitypattern, {
        plot(interval, steps, type="l",ylab="Steps",
             xlab="5-minute Interval Number")
})
```

![](./PA1_template_files/figure-html/timeseries-1.png) 

The interval with the maximum number of average steps is:

```r
subset(activitypattern, steps==max(activitypattern$steps))
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values
Check for the number of rows in the data with missing values:

```r
nrow(activity[!complete.cases(activity),])
```

```
## [1] 2304
```

Replace the missing values with the average from all days for each
five-minute interval.

```r
library(plyr)
NAreplace<-join(activity[!complete.cases(activity),], activitypattern, 
type="left", by="interval")
completeactivity<-activity
completeactivity[!complete.cases(completeactivity),1]<-NAreplace[,4]
```

To calculate the total number of steps taken per day, aggregate the recorded
measurements from all of the 5 minute intervals for each day. 

```r
completedailysteps<-aggregate(steps~date, completeactivity, sum)
```

The following histogram organizes the total steps taken each day into 
value bins:

```r
hist(completedailysteps$steps, col="red", xlab="Total Daily Steps", 
     ylab="Number of Days", main="Total Number of Steps Taken Per Day")
```

![](./PA1_template_files/figure-html/completehistogram-1.png) 

The **mean** number of steps taken per day is:

```r
mean(completedailysteps$steps)
```

```
## [1] 10766.19
```
The **median** number of steps taken per day is:

```r
median(completedailysteps$steps)
```

```
## [1] 10766.19
```

There is no difference in the mean number of steps taken each day with the 
5 minute interval mean substitution. 

```r
mean(completedailysteps$steps)-mean(dailysteps$steps)
```

```
## [1] 0
```

The median of the data increases with substitution of the 5 minute interval mean 
for the NA values:

```r
median(completedailysteps$steps)-median(dailysteps$steps)
```

```
## [1] 1.188679
```

## Are there differences in activity patterns between weekdays and weekends?

To determine whether there are differences in activity patterns between 
weekdays and weekends, the dates provided in the data are reclassified as 
either "weekday" or "weekend. Then the number of steps taken in each interval is
averaged for weekdays and weekends. The resulting plot is shown below:


```r
activityDOW<-completeactivity # Use the dataset with filled in missig values
#Determine which day of the week each date corresponds to
DOW<-weekdays(as.POSIXct(activityDOW$date))
#Create a new weekday-weekend variable in the data by replacing days with a 
#weekday or weekend classification
activityDOW$DOW<-gsub("Saturday|Sunday","Weekend",gsub("Monday|Tuesday|Wednesday|Thursday|Friday","Weekday"
,DOW)) 
# Find the mean steps taken in each interval for weekday and weekend
meanactivityDOW<-aggregate(steps~interval+DOW, activityDOW, mean)
#Using the averaged steps by interval plot Weekday vs Weekend
library(ggplot2)
qplot(interval,steps, data=meanactivityDOW, geom="line", 
main="Average Steps by Interval--Weekday vs Weekend", xlab="interval",ylab="steps") +facet_wrap(~DOW,nrow=2)
```

![](./PA1_template_files/figure-html/weekdayweekend-1.png) 

The plot shown above suggests that people are less active early in
the day on the weekend. There is still a spike in activity, but the maximum 
number of steps taken in any 5 minute interval is lower for weekends than 
weekdays. On weekends, after the early spike in activity, participants in this 
study tend to take more steps per 5 minute interval throughout the rest of the 
day than they do on weekdays.
