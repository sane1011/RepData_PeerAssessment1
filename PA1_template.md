rmarkdown:: render("PA1_template.Rmd", output_format = "html_document")
Assignment Project 1
====================
##Loading and preprocessing data
Assuming that the dataset is already unzipped in the working directory.Loading the data into current session and checking the summary.
```{r}
activity <- read.csv("activity.csv")
str(activity)
```
Coercing the class of the date variable from factor to date to facilitate future analysis.
```{r}
activity$date <- as.Date(activity$date)
```
##Mean total number of steps taken per day
Calculating the total number of steps taken per day and plotting it
```{r,fig.height =4,fig.width=4}
totalSteps <- tapply(activity$steps,activity$date,sum,na.rm=TRUE)
hist(totalSteps,col = "grey",main = "Total Steps taken per Day",xlab = "Total steps by date")
```
Following are the mean and median values in the same order
```{r}
mean(totalSteps,na.rm = TRUE)
median(totalSteps, na.rm = TRUE)
```

##Average daily activity pattern 
Time series plot of the 5 minute interval and the average number of steps taken, avaraged across all days
```{r,fig.height =4,fig.width=4}
avg_by_int <- tapply(activity$steps,activity$interval,mean,na.rm=TRUE)
plot(row.names(avg_by_int),avg_by_int,type="l",xlab="Time intervals (in minutes", ylab = "Average of Total Steps", main = "Time Series Plot of the average of Total Steps in a Day")
```
From the plot we see that the peak lies somewhere between 500-100 
```{r}
max_int <- max(avg_by_int)
x <- match(max_int,avg_by_int)
avg_by_int[x]
```
It can be seen that the 5-minute interval that contains the maximum number of steps is the time interval **835** with the corrosponding frequency *206.1698*
##Imputing missing values
Calculating the total number of missing values.
```{r}
sum(is.na(activity))
```
Replacing the Missing values by the mean of total steps taken per day. 
```{r}
avg <- avg_by_int
x <- which(is.na(activity))
new_activity <- activity
y <- new_activity$steps
y[x] <- avg
new_activity$steps <- y
```
Thus this new_activity dataset contains the imputed values as mean.
Plot of total steps taken per day with missing values imputed
```{r,fig.height =4,fig.width=4}
totalSteps_new <- tapply(new_activity$steps,new_activity$date,sum)
hist(totalSteps_new,fill = "grey",main = "Total Steps taken per Day",xlab = "Total steps by date")
```
Calculating Mean and Median with imputed data
```{r}
mean(totalSteps_new)
median(totalSteps_new)
```

Where previously with missing values removed the data was skewed to the left.With imputed data the graph more closely resembels a normal distribution .Imputed data gives a more accurate discription.
Intrestingly the mean and median of the imputed data are the same.

##Activity patterns between weekdays and weekends
Now to study the activity pattern across weekdays and weekends, We generate two factor variables "weekends" and "weekdays" to seggregate activities accordingly 
```{r}
library(dplyr)
library(plyr)
days <- weekdays(new_activity$date)
new_activity <- cbind(new_activity,days)
new_activity$days <- revalue(new_activity$days,c("Monday"="weekday","Tuesday"="weekday","Wednesday"="weekday","Thursday"="weekday","Friday"="weekday"))
new_activity$days <- revalue(new_activity$days,c("Saturday"="weekend","Sunday"="weekend"))
```
The new variable days is a factor variable with two levels - "weekdays" and "weekend" indicating whether a given date is a weekday or weekend.



```{r,fig.height =4,fig.width=4}
new_avg_by_int <- tapply(new_activity$steps,list(new_activity$interval,new_activity$days),mean)
                         library(reshape2)
new_avg_by_int <- melt(new_avg_by_int)
colnames(new_avg_by_int) <- c("interval","day","steps")
library(lattice)
xyplot(new_avg_by_int$steps ~ new_avg_by_int$interval | new_avg_by_int$day, layout=c(1,2),type="l",main="Time Series Plot of the Average of Total Steps (weekday vs. weekend)",xlab="Time intervals (in minutes)",ylab="Average of Total Steps")