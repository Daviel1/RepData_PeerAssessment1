---
title: 'Reproducible Research: Course Project 1'
author: "David Li"
date: "18 March 2019"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```


## Reproducible Research: Course Project 1
To start, I load the libraries that I will be using for the project.

```{r}
library(dplyr)
library(ggplot2)

```

###Loading and preprocessing the data
####Show any code that is needed to
####1.Load the data (i.e. \color{red}{\verb|read.csv()|}read.csv())

In order to load the data, I use the load.csv function
```{r}
activitydata <-read.csv("activity.csv")
```

To get an idea of the nature of data I apply several functions:
```{r}
str(activitydata)
summary(activitydata)
head(activitydata)
```

####2.Process/transform the data (if necessary) into a format suitable for your analysis
We will need to remove the missing values, so I create a second dataset: 
```{r}
acttransformed <- na.omit(activitydata)
```

###What is mean total number of steps taken per day?
####For this part of the assignment, you can ignore the missing values in the dataset.
####1.Calculate the total number of steps taken per day
Using the dplyr librbary, I subset the dataset by day, aggregating the steps
```{r}
actday <- group_by(acttransformed, date)
actday <- summarise(actday, steps=sum(steps))
```

####2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
```{r}
ggplot(actday, aes(x=steps)) + geom_histogram()
```


####3.Calculate and report the mean and median of the total number of steps taken per day

I apply the mean() and median() functions to calculate the mean and median
```{r}
mean(actday$steps)
median(actday$steps)
```

###What is the average daily activity pattern?
####1.Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
I transform activity data into aggregated averages per 5min interval
```{r}
actint <- group_by(acttransformed, interval)
actint <- summarize(actint, steps=mean(steps))
```
Now I plot the average steps per day on the 5 minute intervals
```{r}
ggplot(actint, aes(interval, steps)) + geom_line()
```

####2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
I find the row of the interval dataset for which the number of steps is the maximum.
```{r}
actint[actint$steps==max(actint$steps),]
```

###Imputing missing values
####Note that there are a number of days/intervals where there are missing values (coded as \color{red}{\verb|NA|}NA). The presence of missing days may introduce bias into some calculations or summaries of the data.
####1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with \color{red}{\verb|NA|}NAs)
We can calcualte the number of missing values by subtracting the original dataset with the omitted missing values dataset
```{r}
nrow(activitydata)-nrow(acttransformed)
```

####2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
Since several days do not have any data, it is not feasible to replace missing values with the day's mean. Instead imputation is achieved through the mean steps per interval across all days. I merge the dataset with this mean with the original data.

```{r}
names(actint)[2] <- "meansteps"
actimpute <- merge(activitydata, actint)
```

####3.Create a new dataset that is equal to the original dataset but with the missing data filled in.
If steps is a missing value, I replace the value with the mean number of steps for the interval:
```{r}
actimpute$steps[is.na(actimpute$steps)] <- actimpute$meansteps[is.na(actimpute$steps)]
```

####4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
I create a new dataset with the total number of steps using imuted data:
```{r}
actdayimputed <- group_by(actimpute, date)
actdayimputed <- summarize(actdayimputed, steps=sum(steps))
```
Then I generate a ggplot to create a histogram and calcualte the mean and median.
```{r}
ggplot(actdayimputed, aes(x=steps)) + geom_histogram()
mean(actdayimputed$steps)
median(actdayimputed$steps)
```

###Are there differences in activity patterns between weekdays and weekends?
####For this part the \color{red}{\verb|weekdays()|}weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.
####1.Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

I first convert the date variable into date class, then I apply the weekday function to create the day for each day of the week. Finally I create a factor variable to distinguish the weekend days.
```{r}
actimpute$dayofweek <- weekdays(as.Date(actimpute$date))
actimpute$weekend <-as.factor(actimpute$dayofweek=="Saturday"|actimpute$dayofweek=="Sunday")
levels(actimpute$weekend) <- c("Weekday", "Weekend")
```


####2.Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
I first create two datasets for weekday and weekend
```{r}
actweekday <- actimpute[actimpute$weekend=="Weekday",]
actweekend <- actimpute[actimpute$weekend=="Weekend",]
```
For each dataset, I calculate the mean steps for each interval
```{r}
actintweekday <- group_by(actweekday, interval)
actintweekday <- summarize(actintweekday, steps=mean(steps))
actintweekday$weekend <- "Weekday"
actintweekend <- group_by(actweekend, interval)
actintweekend <- summarize(actintweekend, steps=mean(steps))
actintweekend$weekend <- "Weekend"
```
Finally, I combine the two datasets and generate two sets of time series plots
```{r}
actint <- rbind(actintweekday, actintweekend)
actint$weekend <- as.factor(actint$weekend)
ggplot(actint, aes(interval, steps)) + geom_line() + facet_grid(weekend ~ .)
```
