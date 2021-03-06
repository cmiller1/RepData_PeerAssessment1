---
title: "RepData_PeerAssessment1"
author: "cmiller1"
date: "February 15, 2015"
output: html_document
---
The assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The variables included in this dataset are:  
* steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)  
* date: The date on which the measurement was taken in YYYY-MM-DD format  
* interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

##Loading and preprocessing the data
First, we'll read in the data into a R DF. To be thorough, the zip file is downloaded from the internet and the uncompressed into an R DF. [The data are downloaded from here](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)The necessary librarys are also loaded:

```r
library(dplyr)
library(ggplot2)
if(!file.exists("temp.zip")) download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip","temp.zip", method="curl", mode="wb")
data0<-read.csv(unzip("temp.zip"))
```

##What is mean total number of steps taken per day?
1. First we'll group the data by date using ddplyr and then summarize the data using the sum of steps:

```r
by_day<-group_by(data0,date)
sum_steps<-summarise(by_day, sum(steps))
```

The total steps taken per day then is shown in the DF:

```r
head(sum_steps)
```

```
## Source: local data frame [6 x 2]
## 
##         date sum(steps)
## 1 2012-10-01         NA
## 2 2012-10-02        126
## 3 2012-10-03      11352
## 4 2012-10-04      12116
## 5 2012-10-05      13294
## 6 2012-10-06      15420
```

2. Now we'll create the histogram using ggplot2's qplot function. For easier viewing, we'll define a bin width of 1000 and make the column names more readable:

```r
names(sum_steps)<-c("date","steps")
qplot(steps, data=sum_steps, geom="histogram", binwidth=1000)
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

3. To report on the mean and median values, we'll just print the summary of the grouped data. The median is 10765 and the mean is 10766.

```r
summary(sum_steps)
```

```
##          date        steps      
##  2012-10-01: 1   Min.   :   41  
##  2012-10-02: 1   1st Qu.: 8841  
##  2012-10-03: 1   Median :10765  
##  2012-10-04: 1   Mean   :10766  
##  2012-10-05: 1   3rd Qu.:13294  
##  2012-10-06: 1   Max.   :21194  
##  (Other)   :55   NA's   :8
```

##What is the average daily activity pattern?
1. First we'll re-group the data by interval and then summarize using the average number of steps taken. Then we'll make a time-series graph of this data:

```r
by_int<-group_by(data0,interval)
avg_int<-summarise(by_int, mean(steps, na.rm=T))
names(avg_int)<-c("interval","mean_all_days")
ggplot(avg_int, aes(x=interval, y=mean_all_days)) + geom_line()
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

*Note: Intervals are kept in original format of dataset so x axis labels refere to interval names and not the actual minutes of the day.*

2. Now we'll select the interval with the highest mean number of steps which is 206.1698 and occurs at interval 835. It is interesting to note that the highest mean number of steps during any single interval is 206, while the highest individual value is 806. This suggests a high variation in daily activity patterns. 


```r
avg_int[which.max(avg_int$mean_all_days),]
```

```
## Source: local data frame [1 x 2]
## 
##   interval mean_all_days
## 1      835      206.1698
```

##Imputing missing values
1. To get the total number of rows with NA values for the steps, we can just refer to the summary and see that it is 2304 rows with NA values:

```r
summary(data0)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

2. My strategy to replace these NAs will be to use the average value for the interval over the entire two month period. This was previously computed and stored in the avg_int DF:

```r
head(avg_int)
```

```
## Source: local data frame [6 x 2]
## 
##   interval mean_all_days
## 1        0     1.7169811
## 2        5     0.3396226
## 3       10     0.1320755
## 4       15     0.1509434
## 5       20     0.0754717
## 6       25     2.0943396
```

3. The following code creates a new DF copy of the old DF and then loops through the DF looking for NA values and when it finds one, it replaces the value with the average for the same time interval over all days.


```r
data1<-data0
for (n in 1: nrow(data1)){
if(is.na(data1[n,1])){
k<-avg_int[avg_int$interval==data1[n,3],2]
data1[n,1]<-k
}
}
```

4. Now we'll look at the new histogram and mean/median data for this updated data set: 

```r
by_day1<-group_by(data1,date)
sum_steps1<-summarise(by_day1, sum(steps))
names(sum_steps1)<-c("date","steps")
qplot(steps, data=sum_steps1, geom="histogram", binwidth=1000)
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 

```r
summary(sum_steps1)
```

```
##          date        steps      
##  2012-10-01: 1   Min.   :   41  
##  2012-10-02: 1   1st Qu.: 9819  
##  2012-10-03: 1   Median :10766  
##  2012-10-04: 1   Mean   :10766  
##  2012-10-05: 1   3rd Qu.:12811  
##  2012-10-06: 1   Max.   :21194  
##  (Other)   :55
```

We can observe from this comparison that there is relatively little change. There were previously only 8 days with NA values so this procuedure has replaced those 8 values with the overall averages. Since my methodology uses mean values to replace NA values, the median value only changes by a single step from 10765 to 10766. The mean value stays exactly the same (as we would expect when using the mean replacement approach). The histogram makes the changes more visible, particularly on a narrower binwidth. 

##Are there differences in activity patterns between weekdays and weekends?
1. We'll create a new factor variable in the dataset with NAs removed to specify if the day is a weekday or weekend. First we need to put the date into the proper date format then we use an if loop to code weekdays with 1 and weekends with 0. The final step is to map the new factor variable on this data:

```r
data1$ndate <- as.Date(data1$date, format="%Y-%m-%d")
data1$weekday <- 1
for (n in 1: nrow(data1)){
if (weekdays(data1[n,4])=="Saturday" | weekdays(data1[n,4])=="Sunday"){
data1[n,5]<-0}
}
data1$day<-factor(data1$weekday, labels=c("weekend","weekday"))
```
Here is a sample of what this looks like:

```r
data1[sample(nrow(data1),10),]
```

```
##          steps       date interval      ndate weekday     day
## 822   36.00000 2012-10-03     2025 2012-10-03       1 weekday
## 7048   0.00000 2012-10-25     1115 2012-10-25       1 weekday
## 10232  0.00000 2012-11-05     1235 2012-11-05       1 weekday
## 15017  0.00000 2012-11-22      320 2012-11-22       1 weekday
## 5016   0.00000 2012-10-18      955 2012-10-18       1 weekday
## 10855  0.00000 2012-11-07     1630 2012-11-07       1 weekday
## 188   65.32075 2012-10-01     1535 2012-10-01       1 weekday
## 11364 31.94340 2012-11-09     1055 2012-11-09       1 weekday
## 11746 99.45283 2012-11-10     1845 2012-11-10       0 weekend
## 7684   0.00000 2012-10-27     1615 2012-10-27       0 weekend
```

2. Finally we'll make a panel time series plot comparing the avg number of steps at each interval on the weekdays vs the weekends:

```r
by_weekday<-group_by(data1,interval, day)
avg_weekday<-summarise(by_weekday, mean(steps))
names(avg_weekday)<-c("interval","day","steps")
ggplot(avg_weekday, aes(x=interval, y=steps))+ geom_line() + facet_grid(day ~ .) + geom_line(stat = "hline", yintercept = "mean")
```

![plot of chunk unnamed-chunk-14](figure/unnamed-chunk-14-1.png) 

Since we have also mapped the average for each panel, we can observe that the average number of steps is slightly higher on the weekends, although not by much. 
