# Reproducible Research: Peer Assessment 1
* * *
#####Author: *Robert A. Lupo*
#
#####Date: *Friday, October 17, 2014*
* * * 
**General settings:**

  * Load data without preprocessing: Data will contain NA values
  * Always make code visible: echo = TRUE
  * **PA** refers to Personal Activity

* * * 
### Loading and preprocessing the data
1. Data file is assumed to be in current directory
2. Attribute classes are set 


```r
raw_Data <- read.csv("activity.csv",sep=",",header=TRUE,
                      colClasses=c("integer","Date","factor"))
```
### What is the mean total number of steps taken per day?
1. NA values are to be ignored
2. Remove the NA values
3. Check removal
4. Graph means using the ggplot2 Library
5. Calculate and report the mean and median total number of steps taken per day


```r
# 1. 
# First 3 rows of raw data
head(raw_Data,3)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
```

```r
# 2.
PA_Data  <- na.omit(raw_Data)
# First 3 rows with NAs removed

# 3.
head(PA_Data,3)
```

```
##     steps       date interval
## 289     0 2012-10-02        0
## 290     0 2012-10-02        5
## 291     0 2012-10-02       10
```

```r
# 4.
library(ggplot2)
ggplot(PA_Data, aes(date, steps)) + 
   geom_bar(stat = "identity", color = "green", fill = "green", width = 0.7) + 
  labs(title = "Mean Total Number of Steps Taken Each Day", 
       x = "Date", y = "Total number of steps")
```

![plot of chunk RR_PA2](./PA1_template_files/figure-html/RR_PA2.png) 

```r
# 5.
avgDailySteps<-aggregate(PA_Data$steps,list(Date=PA_Data$date),FUN = "sum")$x
mean(avgDailySteps)
```

```
## [1] 10766
```

```r
median(avgDailySteps)
```

```
## [1] 10765
```
### What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
# 1.
avgSteps<-aggregate(PA_Data$steps,list(interval=as.numeric(as.character(PA_Data$interval))),FUN=mean)
names(avgSteps)[2] <- "meanSteps"

## time series plot
ggplot(avgSteps, aes(interval, meanSteps)) + geom_line(color = "green", size = 0.8) + labs(title = "Time Series Plot: 5-minute Interval", x = "5 minute intervals", y = "Average Number of Steps")
```

![plot of chunk RR_PA_3](./PA1_template_files/figure-html/RR_PA_3.png) 

```r
# 2.Interval with maximum number of steps
avgSteps[avgSteps$meanSteps==max(avgSteps$meanSteps), ]
```

```
##     interval meanSteps
## 104      835     206.2
```
#### Inputing missing values
1. Calculate and report number of missing (NA) values 
2. Replace NA values with the mean for that day
3. Create a new dataset with NAs replaced
4. Graph a histogram of the total number of steps taken each day 
   using the ggplot2 Library
*4a.Do these values differ from the estimates from the first part of assignment? 
*4b.What is the impact of imputing missing data on the estimates of 
    the total daily number of steps?


```r
# 1.
# Check number of NAs in dataset
sum(is.na(raw_Data))
```

```
## [1] 2304
```

```r
# 2.
completedData <- raw_Data

for (i in 1:nrow(completedData)) {
       if (is.na(completedData$steps[i])) {
           completedData$steps[i] <- avgSteps[which(completedData$interval[i] == 
                                     avgSteps$interval),]$meanSteps
         }
}

# 3.
# Check number of NAs in dataset
sum(is.na(completedData))
```

```
## [1] 0
```

```r
# 4.
ggplot(completedData, aes(date, steps)) + 
       geom_bar(stat = "identity", color = "green",fill = "green", width = 0.7)+ 
       labs(title = "Total Number of Steps Taken Each Day", 
       x = "Date", y = "Total number of steps")
```

![plot of chunk RR_PA_4](./PA1_template_files/figure-html/RR_PA_4.png) 

```r
# 4a. Comparison of both data sets
completedData_TotalSteps<-aggregate(completedData$steps,
                          list(Date=completedData$date),FUN = "sum")$x

# 'Means' comparison of first data set to second (replaced NAs)
mean(completedData_TotalSteps)
```

```
## [1] 10766
```

```r
mean(avgDailySteps)
```

```
## [1] 10766
```

```r
#  No difference
mean(completedData_TotalSteps) - mean(avgDailySteps)
```

```
## [1] 0
```

```r
# 'Medians' comparison of first data set to second (replaced NAs)
median(completedData_TotalSteps)
```

```
## [1] 10766
```

```r
median(avgDailySteps)
```

```
## [1] 10765
```

```r
# Difference = 
median(completedData_TotalSteps) - median(avgDailySteps)
```

```
## [1] 1.189
```
### Are there differences in activity patterns between weekdays and weekends?

1. Convert 'date' factor variables to corresponding weekday/weekend factors
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 
   5-minute interval (x-axis) and the average number of steps taken, 
   averaged across all weekday days or weekend days (y-axis). 
   

```r
# 1. "%A" represents an unabbreviated weekday - Ex: Monday
completedData$weekdays <- factor(format(completedData$date, "%A"))

# Date factors
levels(completedData$weekdays)
```

```
## [1] "Friday"    "Monday"    "Saturday"  "Sunday"    "Thursday"  "Tuesday"  
## [7] "Wednesday"
```

```r
# Converted to "weekday" and "weekend"
levels(completedData$weekdays) <- list(weekday = c("Monday", "Tuesday",
       "Wednesday", "Thursday", "Friday"), weekend = c("Saturday", "Sunday"))
       
# Check Date factors       
levels(completedData$weekdays)
```

```
## [1] "weekday" "weekend"
```

```r
# 2.Panel plot
library(lattice)
tsAvgSteps <- aggregate(completedData$steps, 
            list(interval = as.numeric(as.character(completedData$interval)), 
            weekdays = completedData$weekdays), FUN = "mean")
                      
names(tsAvgSteps)[3] <- "meanSteps"

xyplot(tsAvgSteps$meanSteps ~ tsAvgSteps$interval | tsAvgSteps$weekdays, 
       layout = c(1, 2), type = "l", xlab = "Interval", ylab ="Number of steps")
```

![plot of chunk RR_PA_5](./PA1_template_files/figure-html/RR_PA_5.png) 


