# Reproducible Research: Peer Assessment 1
ppss85  
5 January 2016  





## 1. Introduction

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

A few libraries are needed to run the analysis in particular the plyr, ggplot2 and lubridate packages. Please make sure you have these packages downloaded. The code will laod the libraries automatically. 

```
## Warning: package 'lubridate' was built under R version 3.2.3
```

```
## 
## Attaching package: 'lubridate'
## 
## The following object is masked from 'package:plyr':
## 
##     here
```









## 2. Loading and preprocessing the data

### 2.1 Loading the data


```r
# Encode date of the first download on which results are based
dateDownload <- "January 6, 2016 at 7:41"
```

We make use of the data available at the following URL: https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip. 

The code checks for the existence of the data file within the analysis directory and decides to re-download the package if necessary. If not, the download date of the data being used is January 6, 2016 at 7:41.


```r
# If the file does not exist, retrieve it from Url
if (!(file.exists("activity.csv"))){
  cat("Downloading data file...")
  fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
  download.file(fileUrl,destfile="./activity.zip",method="curl")
  dateDownload <- date() # overwrite date tag for the download
  unzip("./activity.zip")
}
# Load the data into a data.frame
activ <- read.csv2("./activity.csv",sep=",")
```

The dowload date of the data file being used is the following: January 6, 2016 at 7:41.

### 2.2 Inspection and pre-processing of the data


```r
# print summary of data frame
str(activ)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
# save dimension of data frame
nrows <- nrow(activ); ncols <- ncol(activ)
# count number of missing values in each column
miss <- colSums(is.na(activ))
```

A summary of the available data set can be seen below, where we can see that we have 17568 observations of 3 measured variables: "steps", "date" and "interval". We notice that there are a number of missing values for the "step" measurement, exactly 2304. There are 0 missing values for the date and interval variables. We will adress the problem of missing values later in the assignment (see section 3.3)

Printing the output of more than 1 hours of elapsed time (that is at least 13 time intervals), we observe the strange feature that the time interval jumps to 100 after the end of the minute 55 interval.


```r
# plot transition from first to second hour
activ[11:15,]
```

```
##    steps       date interval
## 11    NA 2012-10-01       50
## 12    NA 2012-10-01       55
## 13    NA 2012-10-01      100
## 14    NA 2012-10-01      105
## 15    NA 2012-10-01      110
```

The same pattern is repeated after each hour, e.g. for the thrid to fourth elapsed hour, as printed below. 


```r
# plot transition from third to fourth hour
activ[35:39,]
```

```
##    steps       date interval
## 35    NA 2012-10-01      250
## 36    NA 2012-10-01      255
## 37    NA 2012-10-01      300
## 38    NA 2012-10-01      305
## 39    NA 2012-10-01      310
```


```r
# check number of observation per day
cnt <- count(activ, "date")
n <- mean(cnt$freq)
```

One could think these are simply non-existing measurements but we verify that we have 288 observations per day which corresponds to 24 hours divided in steps of 5 minutes. We rapidly see that we are actually dealing with a coerced time stamp where a pattern like "5"" corresponds in fact to "00:05"" hours and minutes and "245"" to "02:45". To avoid any misinterpreation in the data analysis, it is important we format this interval variable to a more usable format. We already cast the date column to a date format for easier handling in the future. We then calculate the exact elapsed time for each time interval, that is "5" is five minutes elapsed since 00:00, and 145 is 105 minutes elapsed since 00:00. We add this variable to our data frame. We also create a variable corresponding to the associated time in the day in hours and minutes.


```r
# set date column to a date format
activ$date <- as.Date(activ$date,"%Y-%m-%d",stringAsFactors=FALSE)
activ$elapsedminutes <- activ$interval-((activ$interval%/%100)*100)+(activ$interval%/%100)*60
activ$time <- paste(activ$interval%/%100,activ$interval - (activ$interval%/%100)*100,sep=":")
activ$time <- as.factor(activ$time)
#activ$time <- as.Date(activ$time,"%H:%M",stringAsFactors=FALSE)
```

Our new working data set takes the form below:

```r
head(activ,10)
```

```
##    steps       date interval elapsedminutes time
## 1     NA 2012-10-01        0              0  0:0
## 2     NA 2012-10-01        5              5  0:5
## 3     NA 2012-10-01       10             10 0:10
## 4     NA 2012-10-01       15             15 0:15
## 5     NA 2012-10-01       20             20 0:20
## 6     NA 2012-10-01       25             25 0:25
## 7     NA 2012-10-01       30             30 0:30
## 8     NA 2012-10-01       35             35 0:35
## 9     NA 2012-10-01       40             40 0:40
## 10    NA 2012-10-01       45             45 0:45
```







## 3. Analysis

In this section, we provide the different material to answer the questions of the assignment. 

### 3.1 What is mean total number of steps taken per day?

By grouping our data by the date, we can obtain the mean and median number of steps per day. In this first approximation, we neglect the existence of missing values in our data set.


```r
# Summarise the data set by grouping the date variable and summing other variables
stepsumperday <- ddply(activ,~date,summarise,totalsteps=sum(as.numeric(steps),na.rm=TRUE))
# Take the mean or median of the step column
meansteps <- mean(as.numeric(stepsumperday$totalsteps),na.rm=TRUE)
mediansteps <- median(as.numeric(stepsumperday$totalsteps),na.rm=TRUE)
meansteps; mediansteps
```

```
## [1] 9354.23
```

```
## [1] 10395
```

The values obtained for the mean and median number of steps per day are respectively 9.354E+03 and 1.040E+04. As one can see, the mean and the median are not the same which could indicate that this distribution is not normal (not gaussian).This can be appreciated in the figure below that shows the histogram of the mean number of steps taken per day. Of course, the normality could be tested in a quantitative approach (skewness and kurtisis estimation) that however goes beyond the scope of this assignment, at least time wise.


```r
# plot histogram of the total step number
hist(stepsumperday$totalsteps,col=scales::alpha('red',.5),border=F,xlab="Total Steps per Day",main="")
```

<img src="PA1_template_files/figure-html/unnamed-chunk-11-1.png" title="" alt="" style="display: block; margin: auto;" />

It therefore seems at first inspection that our subject has been walking at least 10000 steps a day for almost half of the days during the two months period, rarely more than 15000 steps. A finer binning allows to refine this interpretation.   


```r
# plot histogram of the total step number
hist(stepsumperday$totalsteps,col=scales::alpha('red',.5),border=F,xlab="Total Steps per Day",main="",ylim=c(0,25),xlim=c(0,25000),breaks=10)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-12-1.png" title="" alt="" style="display: block; margin: auto;" />

We now see that there is rather high frequency count in the first bin, that is days with less than 2500 steps, including no steps at all. This bin will clearly pull the mean to lower values with respect to the central maximum content bin. This result will be re-interpreted in section 3.3, in the light of accounting in a reasonnable manner for missing values in the data set. 


### 3.2 What is the average daily activity pattern?

Using the ddply function of the plyr package, we group our data by the time interval variable and we associate to each interval the mean and the standard deviation of the corresponding number of steps, averaged over all days. 


```r
# Take the mean and standard deviation of the number of steps per interval category
plot <- ddply(activ,~elapsedminutes,summarise,meansteps=mean(as.numeric(steps),na.rm=TRUE),std=sd(as.numeric(steps),na.rm=TRUE))
which.max(plot$meansteps); max(plot$meansteps); plot[104,1]
```

```
## [1] 104
```

```
## [1] 206.1698
```

```
## [1] 515
```

The plot below the number of steps for each daily five minute time interval, averaged across al days. We observe that there is barely no walking activity before roughly the 350th time interval and after the 1300th interval. The most natural explanation would be that these periods correspond to the sleeping phase of the subject. 


```r
# simply zoom previous plot by adjusting x-axis plotting range
myxticks <- c(0.,200,400,600,800,1000,1200,1400)
plot(plot$elapsedminutes,plot$meansteps, main="Average daily activity pattern ",xlab="Minutes since midnight",ylab="Mean Number of Steps",type="l",col="blue",xlim=range(myxticks))
grid(NULL,NULL, lwd = 1)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-14-1.png" title="" alt="" style="display: block; margin: auto;" />

We can see that there is strong maximum around close to the 250th minute, at the exact 104 time interval which corresponds to the interval from the 515th elapsed minute to the 520 minute. The maximum number of steps in that time interval is exactly 206.1698113.


```r
# plot a zoom of the time series around maximum 
plot(plot$elapsedminutes,plot$meansteps, main="Average daily activity pattern (zoom)",xlab="Minutes since midnight",ylab="Mean Number of Steps",type="l",col="blue",xlim=c(450,650))
grid(NULL,NULL, lwd = 1)
```

<img src="PA1_template_files/figure-html/unnamed-chunk-15-1.png" title="" alt="" style="display: block; margin: auto;" />

The interpretation, given the data, is that the participant tends to conduct an activity each day that for a short period of time increases his walking rate up to a maximum before gradually decreasing. It exhibits the pattern of a sports activity that starts with a warm-up, a gradual increase in walking speed, and a final cool down phase, like a perhaps a jogging session. 

### 3.3 Imputing missing values

As already mentionned in Section 2, there a number of missing values for the step variabe in the delivered data set. In order to imput for missing values, we have developped the following strategy. At a given time interval, if the steps number is missing, we replace it with the number steps taken for this time interval averaged over all days where it was recorded.


```r
# Group by interval and create a variable "MeanByInterval" with the average step number of each group
# for each observation
activnew <- ddply(activ,.(interval),transform,MeanByInterval= mean(steps,na.rm=TRUE))
activnew$steps <- ifelse(!is.na(activnew$steps), activnew$steps, activnew$MeanByInterval)
```

Based on the new data set with no missing values, we can produce again the histogram of the total number of steps per day, averaged over all days of the data set. For a better understanding of the effect of filling in the missing values, we have overlaid in the plot below the histogram produced in section 3.1 (with missing values) and the new one. 


```r
# Summarise the data set by grouping the date variable and summing other variables
stepsumperdaynew <- ddply(activnew,~date,summarise,totalsteps=sum(as.numeric(steps),na.rm=TRUE))
# Take the mean or median of the step column
meanstepsnew <- mean(as.numeric(stepsumperdaynew$totalsteps),na.rm=TRUE)
medianstepsnew <- median(as.numeric(stepsumperdaynew$totalsteps),na.rm=TRUE)
meanstepsnew
```

```
## [1] 10766.19
```

```r
medianstepsnew
```

```
## [1] 10766.19
```

```r
# plot histogram
par(mfrow=c(1,2))
with(stepsumperdaynew,{
hist(stepsumperday$totalsteps,col=scales::alpha('red',.5),border=F,xlab="Total Steps per Day",ylim=c(0,25),xlim=c(0,25000),breaks=10,main="Missing Values")
hist(stepsumperdaynew$totalsteps,col='skyblue',border=F,xlab="Total Steps per Day",main="Missing Values Replaced",ylim=c(0,25),xlim=c(0,25000),breaks=10)
})
```

<img src="PA1_template_files/figure-html/unnamed-chunk-17-1.png" title="" alt="" style="display: block; margin: auto;" />

One can see that after replacing the missing values, we singificantly alter the shape of the total step number distribution below the central maximum content bin. In particular, we now clearly see that the first bin has much lower frequency which indicates its previous high frequency was the result of the missing values interpreted as 0. We expect this new distribution to have a different mean and median with respect to the set with missing values from section 3.1. In the table below, we summarize these quantities for the two cases explored:

Quantity | Missing Values | Missing Values Replaced
---------|----------------|------------------------
Mean     | 9.354E+03   | 1.077E+04
Median   | 1.040E+04 | 1.077E+04

We now see that after replacing for the missing values in the data set (second column), bith the mean and median are in perfect agreement. 

It is important to consider these results with caution since our choice for replacing the missing values in the data set should require carefull inspection and verification.

### 3.4 Are there differences in activity patterns between weekdays and weekends?

Based on the date information contain in the data set, we can associate to each measurement a "daytype" tag of being a weekday or a weekend day (Saturday or Sunday). We then calculate the mean number steps for each time interval and each datype. 


```r
# Add a column with correspoonding weekday or weekend tag
activnew$daytype <- ifelse(weekdays(activnew$date)=="Saturday" | weekdays(activnew$date)=="Sunday","weekend","weekday")
# we group the set by the "daytype" and "interval" variables and calculate the average steps across all days as well as the associated standard deviation
daypattern <- ddply(activnew,c("daytype","elapsedminutes"),summarise,meansteps=mean(as.numeric(steps),na.rm=TRUE),std=sd(as.numeric(steps),na.rm=TRUE))
head(daypattern,5)
```

```
##   daytype elapsedminutes  meansteps       std
## 1 weekday              0 2.25115304 8.6005313
## 2 weekday              5 0.44528302 2.6789143
## 3 weekday             10 0.17316562 1.0418000
## 4 weekday             15 0.19790356 1.1906286
## 5 weekday             20 0.09895178 0.5953143
```

The plot below shows the time series plot of the 5-minute interval (x-axis) and the average number of steps (y-axis), averaged across all weekday days or weekend days. The average step number was divided by 10 for more clarity. 


```r
ggplot(data = daypattern, aes(x = elapsedminutes, y = meansteps*0.1, color = factor(daytype))) + 
  geom_line() +
  facet_wrap( ~daytype, ncol=1) +
  xlab("Minutes since midnight") +
  ylab(expression(paste("Average Number of Steps x 10"^"-1"))) +
  theme(legend.position = "none") +
  theme(strip.text=element_text(size=12)) +
  theme(axis.title.x = element_text(vjust=-0.5)) +
  theme(axis.title.y = element_text(vjust=+1.5))
```

<img src="PA1_template_files/figure-html/unnamed-chunk-19-1.png" title="" alt="" style="display: block; margin: auto;" />

One can see from this plot that the peak walking acitivity observed in section 3.2 in more significant on weekdays. On weekend days, the avergage number of steps per day is more uniform throughout the day. 

## Discussion

The purpose of this assigment was to illustrate the writing of a document following the principle of reproducible research. This one through a pratical example of performing some exploratory analysis of a data set provided by Coursera, making use of data from a personal activity monitoring device.

Through this work, we have explored simple distributions that allowed us to identify patterns in the walking activity of the subject, like a peak in activity associated with weekdays and apparently not or much less conducted on weekend days.

Clear conclusions must however be accompagnied by a more robust analysis of the data, including an association of error measurement to all averaged variables. This a first step in order to go the way of proving the statistical significance of our preliminary interpretations. 
