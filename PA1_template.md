
# Reproducible Research: Peer Assessment 1




##  Loading and preprocessing the data
1. Load the data
unzip the archive as csv and load it inside 'data':
    
    ```r
    unzip("activity.zip")
    data <- read.csv("activity.csv")
    ```

2. Process data if necessary
Read the data into a tbl_df, readable by dplyr:
    
    ```r
    data_tbl <- tbl_df(data)
    ```


## What is mean total number of steps taken per day?
1. Make a histogram of the total number of steps taken each day
    
    ```r
    by_date <- group_by(data_tbl, date)
    total_steps_by_date <- summarize(by_date, total_steps = sum(steps, na.rm=TRUE))
    qplot(total_steps, data = total_steps_by_date, geom="histogram")
    ```
    
    ![plot of chunk histogram](figure/histogram-1.png) 

2. Calculate and report the **mean** and **median** total number of steps taken per day
Mean:
    
    ```r
    mean(total_steps_by_date$total_steps)
    ```
    
    ```
    ## [1] 9354.23
    ```

Median:
    
    ```r
    median(total_steps_by_date$total_steps)
    ```
    
    ```
    ## [1] 10395
    ```


## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
    
    ```r
    by_interval <- group_by(data_tbl, interval)
    mean_steps_by_interval <- summarize(by_interval, mean_steps = mean(steps, na.rm=TRUE))
    plot(mean_steps_by_interval$interval, mean_steps_by_interval$mean_steps, type="l", xlab="5-minute interval", ylab="Average number of steps taken, averaged across all days")
    ```
    
    ![plot of chunk timeseries](figure/timeseries-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
    
    ```r
    arrange_mean_steps_by_interval <- arrange(mean_steps_by_interval, desc(mean_steps))
    arrange_mean_steps_by_interval[1,1]
    ```
    
    ```
    ## Source: local data frame [1 x 1]
    ## 
    ##   interval
    ## 1      835
    ```


## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
    
    ```r
    sum(is.na(data_tbl[,1]))
    ```
    
    ```
    ## [1] 2304
    ```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
    
    ```r
    mutated_data_tbl <- mutate(by_interval, mean_interval_steps = mean(steps, na.rm=TRUE))
    impute_mutated_data_tbl_unfinished <- within(mutated_data_tbl, {
      steps = as.character(steps)
      mean_interval_steps = as.character(mean_interval_steps)
      steps = ifelse(is.na(steps), mean_interval_steps, steps)
      }
    )
    ```

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
    
    ```r
    impute_mutated_data_tbl_finished <- select(impute_mutated_data_tbl_unfinished, steps:interval)
    ```

4. Make a histogram of the total number of steps taken each day and calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
    
    ```r
    by_date_imputed <- group_by(impute_mutated_data_tbl_finished, date)
    by_date_imputed$steps <- as.numeric(by_date_imputed$steps)
    total_steps_by_date_imputed <- summarize(by_date_imputed, total_steps = sum(steps))
    qplot(total_steps, data = total_steps_by_date_imputed, geom="histogram")
    ```
    
    ![plot of chunk histogramrepeat](figure/histogramrepeat-1.png) 

Mean:

```r
mean(total_steps_by_date_imputed$total_steps)
```

```
## [1] 10766.19
```

Median:

```r
median(total_steps_by_date_imputed$total_steps)
```

```
## [1] 10766.19
```

Both  mean and median are higher values in the samples. Note that in the sample with NAs intact, the imputted sample has the same mean and median values, which is a more normal distribution.



## Are there differences in activity patterns between weekdays and weekends?
1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
    
    ```r
    data_tbl_ww <- mutate(data_tbl, day_of_week = wday(date))
    data_tbl_ww$day_of_week <- gsub(1, "Weekday", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(2, "Weekday", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(3, "Weekday", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(4, "Weekday", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(5, "Weekday", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(6, "Weekend", data_tbl_ww$day_of_week)
    data_tbl_ww$day_of_week <- gsub(7, "Weekend", data_tbl_ww$day_of_week)
    ```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
    
    ```r
    by_interval_ww <- group_by(data_tbl_ww, interval, day_of_week)
    mean_steps_by_interval_ww <- summarize(by_interval_ww, mean_steps = mean(steps, na.rm=TRUE))
    ggplot(mean_steps_by_interval_ww, aes(interval,mean_steps)) + geom_line() + facet_grid(day_of_week ~ .) + xlab("5-minute interval") + ylab("Average number of steps taken")
    ```
    
    ![plot of chunk timeseriesrepeat](figure/timeseriesrepeat-1.png) 


 