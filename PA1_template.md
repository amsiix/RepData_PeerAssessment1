# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
```
if ( !file.exists("activity.zip") ) {
    download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip")
} 

if ( !file.exists("activity.csv") ) {
    unzip("activity.zip")
}
```
Finally, read the data into a data frame.
```
data <- read.csv("activity.csv")
```

### histogram of the total number of steps taken each day.
```
library(ggplot2)
# Remove NAs
steps_taken <- data[ complete.cases(data),]

#aggregate steps taken by day
steps_summarized <- aggregate( steps_taken$steps, by=list(steps_taken$date),FUN=sum)

# Rename columns
colnames(steps_summarized) <- c("date","steps")

#Plot the histogram
ggplot(data=steps_summarized, aes(x=steps)) + geom_histogram(binwidth=1000) + xlab("Number of Steps Taken Per Day") 
```
## What is mean total number of steps taken per day?

### Mean number of steps 
```
mean(steps_summarized$steps)
```
### Median number of steps taken per day
```
median(steps_summarized$steps)
```

## What is the average daily activity pattern?
```
average_activity <- aggregate( steps_taken$steps,by=list(steps_taken$interval),FUN=mean)
colnames(average_activity) <- c("interval","average_steps")
ggplot(data=average_activity,aes(x=interval,y=average_steps))+geom_line() + xlab("5 Minute Interval Index") + ylab("Average Number of Steps")
```

## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
