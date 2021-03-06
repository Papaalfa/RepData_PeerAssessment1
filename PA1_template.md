# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Uploading the dataset into R as a dataset named "act":

```r
act <- read.csv("activity.csv")
```

Setting the date into a correct format:

```r
act$date <- as.Date(act$date)
```

## What is mean total number of steps taken per day?

By now ignoring the missing values.
Calculating the total number of steps taken per day:

```r
byday <- tapply(act$steps, act$date, sum)
```

Making a histogram of the total number of steps taken each day


```r
hist(byday
     , xlab="Average Number of Steps per Day"
     , main="Number of Steps taken Each Day")
```

![](PA1_template_files/figure-html/hist-1.png)

Calculating and reporting the mean and median of the total number of steps taken per day:

```r
mn <- mean(byday, na.rm = T)
md <- median(byday, na.rm = T)
cat(paste("Mean is:",mn), "\n"
    , paste("Median is:",md))
```

```
## Mean is: 10766.1886792453 
##  Median is: 10765
```

## What is the average daily activity pattern?

Making a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days:

```r
## Computing intervals averages
byint <- with(act, tapply(steps, interval, na.rm=T, mean))
## Ploting a time series
plot(names(byint), byint, type="l"
     , xlab="5 Minute Interval"
     , ylab="Average Number of Steps"
     , main="Number of Steps Taken (Across All Days)")
```

![](PA1_template_files/figure-html/plot-1.png)

Now finding out which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
maxst <- names(byint[byint==max(byint)])
cat(paste("The interval containing the maximum number of steps is:",maxst))
```

```
## The interval containing the maximum number of steps is: 835
```

## Imputing missing values

There are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculating and reporting the total number of missing values in the dataset:

```r
## check NAs number and share
total <- sum(is.na(act$steps))
share <- mean(is.na(act$steps))
cat(paste("Total number of missing values is:",total),"\n"
    , paste0("Share of missing values in the dataset is: ",round(share*100),"%"))
```

```
## Total number of missing values is: 2304 
##  Share of missing values in the dataset is: 13%
```

From my perspective the best strategy of imputing the NAs in the case is replacing them by average values of the 5-minute intervals.

So now I'm creating a new dataset ("act2") that is equal to the original dataset but with the missing data filled in:

```r
act2 <- transform(act, steps = ifelse(is.na(steps), tapply(steps, interval, na.rm=T, mean), steps))
```

Making a histogram of the total number of steps taken each day:

```r
byday2 <- tapply(act2$steps, act2$date, sum)
hist(byday2
     , xlab="Average Number of Steps per Day"
     , main="Number of Steps taken Each Day (NAs removed)")
```

![](PA1_template_files/figure-html/hist2-1.png)

Calculating and reporting the mean and median total number of steps taken per day:

```r
mn2 <- mean(byday2, na.rm = T)
md2 <- median(byday2, na.rm = T)
cat(paste("Mean with NAs is:",mn), "\n"
    , paste("Median with NAs is:",md))
```

```
## Mean with NAs is: 10766.1886792453 
##  Median with NAs is: 10765
```

```r
cat(paste("Mean (NAs removed) is:",mn2), "\n"
    , paste("Median (NAs removed) is:",md2))
```

```
## Mean (NAs removed) is: 10766.1886792453 
##  Median (NAs removed) is: 10766.1886792453
```

We can see that now the mean is the same, but the median is a slightly bigger number.
So what is the impact of imputing missing data on the estimates of the total daily number of steps?
To figure this out I plot histograms for both cases side by side:

```r
mfrow <- par("mfrow")
par(mfrow = c(1,2))
hist(byday, ylim=c(0,40)
     , xlab="Average Number of Steps per Day"
     , main="With NAs")
hist(byday2, ylim=c(0,40)
     , xlab=""
     , ylab=""
     , main="NAs removed")
```

![](PA1_template_files/figure-html/unnamed-chunk-9-1.png)

```r
## Set the mfrow back to default
par(mfrow = mfrow)
```

Now we can see that the most frequent part of the histogram is bigger for the "NAs removed" case, which makes sense as I've filled the NAs with the average values.

## Are there differences in activity patterns between weekdays and weekends?

For this part I use the dataset with the filled-in missing values.
First I create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day. I also use the [dplyr package](https://cran.r-project.org/web/packages/dplyr/index.html) to manipulate the dataset and [lattice package](https://cran.r-project.org/web/packages/lattice/index.html) to make the plot. 

```r
## Here I have to set the locale to English as my default is Russian
mylocale <- Sys.getlocale("LC_TIME")
Sys.setlocale("LC_TIME","English")
```

```
## [1] "English_United States.1252"
```

```r
## Load the dplyr package
suppressMessages(require(dplyr))

actwd <- act2 %>% 
    mutate(wd=factor(ifelse(weekdays(as.Date(date)) %in% c("Saturday","Sunday")
                            , "weekend"
                            ,"weekday")))
```

Now I make a panel plot containing a time series plot of the 5-minute interval and the average number of steps taken, averaged across all weekday days or weekend days.

```r
## Make a suumary dataset for the plot
avgwd <- actwd %>% group_by(interval,wd) %>% summarise(avg=mean(steps,na.rm=T))

## Load the lattice package
suppressMessages(require(lattice))

## Build the plot
xyplot(avg ~ interval | wd, data=avgwd
       , layout=c(2,1)
       , panel = function(x, y) {
           panel.xyplot(x, y, type="l")
           panel.abline(h=mean(y), col="red", lwd=2)
       }
       , xlab="5 Minute Interval"
       , ylab="Average Number of Steps"
       , main="Activity on Weekdays and Weekends")
```

![](PA1_template_files/figure-html/xyplot-1.png)

```r
## Set the local back to default
Sys.setlocale("LC_TIME", mylocale)
```

```
## [1] "Russian_Russia.1251"
```

From the plot we can see there are some differences between weekdays and weekends:  
  1. For weekdays there is a morning peak and the rest of the day is quite flat.  
  2. On weekends the steps number distribution is of a normal shape with more activity in the middle of a day and less in both morning and evening.  
  3. Average number of steps is slightly bigger on weekends.

