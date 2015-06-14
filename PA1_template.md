Reproducible Research Peer Assessment 1
---------------------------------------
PA1_template.Rmd
--------------------------------------
By Jaja Yogo (June, 2015).

---------------------------------------

## Introduction
Data about personal movements using activity monitoring devices like a Fitbit, Nike Fuelband, or Jawbone up remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

The data from a personal activity monitoring device consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

## Data
The data for this assignment can be downloaded from the course web site:
*Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K].
The variables included in this dataset are:
**Steps**: Number of steps taking in a 5-minute interval (missing values are coded as NA).
**Date**: The date on which the measurement was taken in YYYY-MM-DD format
**Interval**: Identifier for the 5-minute interval in which measurement was take.

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Assignment
This assignment will be completed using a **single R markdown** document that can be processed by **knitr** and be transformed into an HTML file.  

## Loading and preprocessing data
Here is a code chunk to load the data file.

```{r echo=TRUE}
rdata <- read.csv('activity.csv', header = TRUE, sep = ",",
                  colClasses=c("numeric", "character", "numeric")) 
```
## Tidy the data
Here the date field is changed to Date class and interval field to Factor class.
```{r} rdata$date <- as.Date(rdata$date, format = "%Y-%m-%d")
rdata$interval <- as.factor(rdata$interval)
```

###Let us confirm the data using 
str() method:
str(rdata) 

### Find the mean total of steps taken per day?
Ignore the missing data (NA). 
Then calculate the steps taken each day.

```{r} 
steps_per_day <- aggregate(steps ~ date, rdata, sum)
colnames(steps_per_day) <- c("date","steps")
head(steps_per_day)
```
```{r} data.ignore.na <- na.omit(data) 

# sum steps by date
daily.steps <- rowsum(data.ignore.na$steps, format(data.ignore.na$date,
'%Y-%m-%d')) 
daily.steps <- data.frame(daily.steps) 
names(daily.steps) <- ("steps") 
```

**Plot a histogram of the total number of steps taken per day, using appropriate bin interval.**

```{r, echo=TRUE} hist(daily.steps$steps, 
     main=" ",
     breaks=10,
     xlab="Total Number of Steps Taken Daily")
 
```

### Find the mean and median total number of steps taken per day
```{r} mean(daily.steps$steps); 
```
```{r} median(daily.steps$steps) 
```

### Find the average daily activity pattern
Calculate average steps for each of 5-minute interval, then convert the intervals as integers, save fiindings in a data franme names "steps-per_interval". 

```{r} library(plyr)
** Calculate average steps for each of 5-minute interval during a 24-hour period.**

```{r}interval.mean.steps <- ddply(data.ignore.na,~interval, summarise, mean=mean(steps))
```
**Plot time series of the 5-minute interval and the average number of steps taken, averaged across all days**

```{r, echo=TRUE} library(ggplot2)
qplot(x=interval, y=mean, data = interval.mean.steps,  geom = "line",
      xlab="5-Minute Interval (military time)",
      ylab="Number of Step Count",
      main="Average Number of Steps Taken Averaged Across All Days"
      )
```
**Provide the 5-min interval, on average across all the days in the dataset, contains the maximum number of steps:**
```{r}interval.mean.steps[which.max(interval.mean.steps$mean), ]
```

#convert to integers
### Does helps in plotting.

```{r} steps_per_interval$interval <- 
        as.integer(levels(steps_per_interval$interval)
        [steps_per_interval$interval])
colnames(steps_per_interval) 
<- c("interval", "steps")
```

### Imputing missing values:
**Note** that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.The total number of missing values in steps can be calculated using is.na() method to check whether the value is missing or not and then summing the logical vector.

**Calculate and report the total number of missing values in the dataset**
(i.e. the total number of rows with NAs).

```{r} library(sqldf)
tNA <- sqldf(' 
    SELECT d.*            
    FROM "data" as d
    WHERE d.steps IS NULL 
    ORDER BY d.date, d.interval ')
    ```
## Loading required package: tcltk
```{r} NROW(tNA) 
```

**Make a new dataset** (t1) that is similar to the original dataset but with the missing data erased. The dataset is ordered by date and interval. The following SQL statement combines the original “data” dataset set and the “interval.mean.steps” dataset that contains mean values of each 5-min interval ageraged across all days. 

```{r} t1 <- sqldf('  
    SELECT d.*, i.mean
    FROM "interval.mean.steps" as i
    JOIN "data" as d
    ON d.interval = i.interval 
    ORDER BY d.date, d.interval ')
    ```

```{r} t1$steps[is.na(t1$steps)] <- t1$mean[is.na(t1$steps)]
In the following, prepare data for plot histogram calculate mean and median:
t1.total.steps <- as.integer( sqldf(' 
    SELECT sum(steps)  
    FROM t1') );
    ```

```{r} t1.total.steps.by.date <- sqldf(' 
    SELECT date, sum(steps) as "t1.total.steps.by.date" 
    FROM t1 GROUP BY date 
    ORDER BY date')
    ``` 

```{r} daily.61.steps <- sqldf('   
    SELECT date, t1_total_steps_by_date as "steps"
    FROM "t1.total.steps.by.date"
    ORDER BY date') 
```
**Make a histogram of the total number of steps taken each day**.
```{r, echo=TRUE} hist(daily.61.steps$steps, 
     main=" ",
     breaks=10,
     xlab="After Imputate NA -Total Number of Steps Taken Daily")
```

```{r}library(plyr)
```
### Calculate average steps for each of 5-minute interval during a 24-hour period
```{r, echo=TRUE}interval.mean.steps <- ddply(data.ignore.na,~interval, summarise, mean=mean(steps))
```

**Next, find the 5- minute interval plus the maximum number of steps**:
```{r, echo+TRUE} max_interval <- steps_per_interval[which.max(  
        steps_per_interval$steps),]
```

### Observed outcome:
**Any difference in values initial and processed values?**
The **mean and median** have same value of **10766.189** afte filling data, as opposed to **10766.189** and **10765** respectively, before filling the data.

## Differences in activity patterns between weekday and weekends
**Create a factor variable weektime with two levels**
(weekday, weekend). The following dataset t5 dataset contains data: average number of steps taken averaged across all weekday days and weekend days, 5-min intervals, and a facter variable weektime with two levels (weekday, weekend).

```{r, echo=TRUE} t1$weektime <- as.factor(ifelse(weekdays(t1$date) %in% 
                c("Saturday","Sunday"),"weekend", "weekday"))
                ```

```{r} t5 <- sqldf('   
    SELECT interval, avg(steps) as "mean.steps", weektime
    FROM t1
    GROUP BY weektime, interval
    ORDER BY interval ')
```
**Make a panel plot containing a time series plot** (i.e. type = "l") of the minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```{r} library(lattice)
```
``` {r , echo=TRUE} p <- xyplot(mean.steps ~ interval | factor(weektime), 
data=t5, type = 'l',
       main="Average Number of Steps Taken 
       \nAveraged Across All Weekday Days or Weekend Days",
       xlab="5-Minute Interval (military time)",
       ylab="Average Number of Steps Taken")
print (p) 
```

### Observed outcome:
Yes, there are differences in activity patterns bewteen weedays and weekends.The plot indicates that the person is more active on the weekend days.

## Conclusion
This assignment provides a student with a systematic approach of conducting research which can be reproducible. If the data and tools of analyses are utilized in acceptable scietific approach.
