# Reproducible-Data_PeerAssessment1
Introduction:
Data about personal movements using activity monitoring devices like a Fitbit, Nike Fuelband, or Jawbone up remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

The data from a personal activity monitoring device consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

Data:
The data for this assignment can be downloaded from the course web site: Dataset: Activity monitoring data [52K]. The variables included in this dataset are: Steps: Number of steps taking in a 5-minute interval (missing values are coded as NA). Date: The date on which the measurement was taken in YYYY-MM-DD format *Interval: Identifier for the 5-minute interval in which measurement was taken.

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

Loading and preprocessing data:
Here is a code chunk to load the data file.

rdata <- read.csv('activity.csv', header = TRUE, sep = ",",
                  colClasses=c("numeric", "character", "numeric")) 
Tidy the data:
Here the date field is changed to Date class and interval field to Factor class. ```{r echo=TRUE} rdatadate<−as.Date(rdatadate, format = “%Y-%m-%d”) rdatainterval<−as.factor(rdatainterval) Let us confirm the data using

{ r echo=TRUE} str() method: str(rdata)

The mean total of steps taken per day?
Ignore the missing data (NA). Then calculate the steps taken each day.

{ r echo=TRUE}  steps_per_day <- aggregate(steps ~ date, rdata, sum) colnames(steps_per_day) <- c("date","steps") head(steps_per_day)

Plot a histogram of the total number of steps taken per day, using appropriate bin interval.

{ r echo=TRUE}  ggplot(steps_per_day, aes(x = steps)) +         geom_histogram(fill = "green", binwidth = 1000) +          labs(title="Histogram of Steps Taken per Day",               x = "Number of Steps per Day", y = "Number of times in a day(Count)") + theme_bw()

Next, find the mean and median total number of steps taken per day { r echo=TRUE} steps_mean   <- mean(steps_per_day$steps, na.rm=TRUE) steps_median <- median(steps_per_day$steps, na.rm=TRUE)

The average daily activity pattern?
Calculate average steps for each of 5-minute interval, then convert the intervals as integers, save fiindings in a data franme names “steps-per_interval”.

{r echo=TRUE} steps_per_interval <- aggregate(rdata$steps,                                  by = list(interval = rdata$interval),                               FUN=mean, na.rm=TRUE) #convert to integers ##this helps in plotting {r echo=TRUE} steps_per_interval$interval <-          as.integer(levels(steps_per_interval$interval)[steps_per_interval$interval]) colnames(steps_per_interval) <- c("interval", "steps")

Plot with time series of average number of steps taken ( on all days) versus the 5-munite:

Imputing missing values:
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.The total number of missing values in steps can be calculated using is.na() method to check whether the value is missing or not and then summing the logical vector. {r echo=TRUE} missing_vals <- sum(is.na(rdata$steps))

library(plyr) # Calculate average steps for each of 5-minute interval during a 24-hour period {r echo=TRUE}interval.mean.steps <- ddply(data.ignore.na,~interval, summarise, mean=mean(steps)) Plot time series of the 5-minute interval and the average number of steps taken, averaged across all days: {r echo=TRUE} ggplot(steps_per_interval, aes(x=interval, y=steps)) +            geom_line(color="red", size=1) +           labs(title="Average Daily Activity Pattern", x="Interval", y="Number of steps") +           theme_bw()

Next, find the 5- minute interval plus the maximum number of steps: {r echo=TRUE} max_interval <- steps_per_interval[which.max(           steps_per_interval$steps),]

Observed outcome:
Differences in activity patterns between weekday and weekends:
Create a factor variable weektime with two levels (weekday, weekend). The following dataset t5 dataset contains data: average number of steps taken averaged across all weekday days and weekend days, 5-min intervals, and a facter variable weektime with two levels (weekday, weekend).

```{r echo=TRUE} t1weektime<−as.factor(ifelse(weekdays(t1date) %in% c(“Saturday”,“Sunday”),“weekend”, “weekday”))

t5 <- sqldf(‘
SELECT interval, avg(steps) as “mean.steps”, weektime FROM t1 GROUP BY weektime, interval ORDER BY interval’) ``` Make a panel plot containing a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

library(lattice) {r echo =TRUE} p <- xyplot(mean.steps ~ interval | factor(weektime), data=t5,         type = 'l',        main="Average Number of Steps Taken         \nAveraged Across All Weekday Days or Weekend Days",        xlab="5-Minute Interval (military time)",        ylab="Average Number of Steps Taken") print (p)

Observed outcome:
Conclusion
