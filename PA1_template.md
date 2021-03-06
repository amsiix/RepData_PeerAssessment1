# Reproducible Research: Peer Assessment 1


This assignment makes use of data from a personal activity monitoring
device. This device collects data at 5 minute intervals through out the
day. The data consists of two months of data from an anonymous
individual collected during the months of October and November, 2012
and include the number of steps taken in 5 minute intervals each day.

### Data

The data for this assignment can be downloaded from the course web
site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing
    values are coded as `NA`)

* **date**: The date on which the measurement was taken in YYYY-MM-DD
    format

* **interval**: Identifier for the 5-minute interval in which
    measurement was taken


The dataset is stored in a comma-separated-value (CSV) file and there
are a total of 17,568 observations in this

## Loading and preprocessing the data


Load the data (i.e. `read.csv()`)


```r
# Download zip file if neccesary
if ( !file.exists("activity.zip") ) {
    download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip")
} 

# Unzip the file if necessary
if ( !file.exists("activity.csv") ) {
    unzip("activity.zip")
}

data <- read.csv("activity.csv")
```

### histogram of the total number of steps taken each day.

Calculate the total number of steps per day.

```r
library(ggplot2)
# Remove NAs
steps_taken <- data[ complete.cases(data),]

#aggregate steps taken by day
steps_summarized <- aggregate( steps_taken$steps, by=list(steps_taken$date),FUN=sum)

# Rename columns
colnames(steps_summarized) <- c("date","steps")
```

Make a histogram of the total number of steps taken each day

```r
#Plot the histogram
ggplot(data=steps_summarized, aes(x=steps)) + 
    geom_histogram(binwidth=1000) + 
    xlab("Number of Steps Taken Per Day")  +
    theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)\
## What is mean total number of steps taken per day?

Calculate and report the **mean** and **median** total number of steps taken per day

### Mean number of steps 

```r
mean(steps_summarized$steps)
```

```
## [1] 10766.19
```
### Median number of steps taken per day

```r
median(steps_summarized$steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?

```r
# Take the average per each interval index
average_activity <- aggregate( steps_taken$steps,by=list(steps_taken$interval),FUN=mean)

# Rename the column names
colnames(average_activity) <- c("interval","average_steps")
```

Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
# Plot the 
ggplot(data=average_activity,aes(x=interval,y=average_steps)) +
    geom_line() + 
    xlab("5 Minute Interval Index") + 
    ylab("Average Number of Steps") +
    theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)\

Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
# Compute the max average steps
max_steps = max( average_activity$average_steps)

# Return the interval(s) that have this max value of average_steps
average_activity[ average_activity$average_steps == max_steps,]
```

```
##     interval average_steps
## 104      835      206.1698
```
## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)

```r
sum(!complete.cases(data))
```

```
## [1] 2304
```

The strategy for filling in all of the missing values in the dataset: substitute the average for the interval. 

Create a new dataset that is equal to the original dataset but with the missing data filled in:

```r
# Copy the original dataset
imputed_data <- data
```


```r
#Add an imputed_steps variable. When
for( thisRecord in 1:nrow(imputed_data)) {
    # if the steps are NA, then impute the value with the average steps, otherwise use the given steps value
    imputed_data$steps[thisRecord] <- 
        ifelse( 
            is.na(imputed_data$steps[thisRecord]), 
                average_activity[ average_activity$interval == imputed_data$interval[thisRecord], "average_steps"],
                imputed_data$steps[thisRecord] 
        )
    
}
```

Total number of rows with `NA`s in imputed dataset

```r
sum(!complete.cases(imputed_data))
```

```
## [1] 0
```

Make a histogram of the total number of steps taken each day

```r
library(ggplot2)
# Remove NAs
#aggregate steps taken by day
steps_summarized_imputed <- aggregate( imputed_data$steps, by=list(imputed_data$date),FUN=sum)

# Rename columns
colnames(steps_summarized_imputed) <- c("date","steps")

#Plot the histogram
ggplot(data=steps_summarized_imputed, aes(x=steps)) + 
    geom_histogram(binwidth=1000) + 
    xlab("Number of Steps Taken Per Day")  +
    theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)\

Calculate and report the **mean** and **median** total number of steps taken per day.

```r
mean(steps_summarized_imputed$steps)
```

```
## [1] 10766.19
```


```r
median(steps_summarized_imputed$steps)
```

```
## [1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? 
The mean is the same. However, the median did move slightly, and coincides with the new mean.

What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
steps_summarized_imputed <- aggregate( imputed_data$steps, by=list(imputed_data$date),FUN=sum)

# Rename columns
colnames(steps_summarized_imputed) <- c("date","steps")

library(ggplot2)
ggplot() + 
    geom_line(data=steps_summarized_imputed,aes(x=date,y=steps, group=1), binwidth=1000,colour="red") +
    geom_line(data=steps_summarized,aes(x=date,y=steps,group=1), binwidth=1000, colour="blue") +
    theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)\


## Are there differences in activity patterns between weekdays and weekends?
Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
# Set the day type
library(lubridate)
imputed_data$day_type <- factor( ifelse( wday(as.Date(imputed_data$date)) %in% c(1,7), "weekend", "weekday" ) )
```

Make a panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
# Summarize average steps by interval and day type
average_activity_by_day_type <- aggregate( imputed_data$steps,by=list(imputed_data$day_type,imputed_data$interval),FUN=mean)
colnames( average_activity_by_day_type ) <- c("day_type","interval", "steps")

# Plot activity by day type
library(ggplot2)
ggplot( data=average_activity_by_day_type, aes(x=interval,y=steps)) +
    geom_line() +
    facet_grid( day_type~. ) +
    theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-18-1.png)\
