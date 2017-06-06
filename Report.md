# Introduction
This project forms part of the Reproducible Research course on Coursera. The goal of this project is to give the student an opportunity to conduct an analysis that is reproducible: "the ability of an entire [study] to be reproduced, etheir by the researcher or by someone else working independently" (Wikipedia).
To this end, the assignment requires students to conduct an analysis and to consequently produce a report by following a literate programming approach, where text and code are weaved toghether in a human-readable format. This type of report will be produced with knitr.
## Analysis
### Data
The data for the analysis is sourced from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November in 2012. The data is available here. The variables included in the data are:
* steps: Number of steps taken in a 5-minute interval
* date: The date on which the measurement was taken in YYYY-MM-DD format
* interval: A unuiqe identifier for the 5-minute interval in which measurement was taken
Missing values are coded as "NA". The dataset contains 17,568 observations and is stored in a comma-sperated value (CSV) format.

### Loading and preprocessing the data
1. Load the data
The data is loaded into a R variable called activity_data:
```
activity_data <- read.csv('./activity.csv')
```
2. Process the data
The variables are converter to their correct representions in R:
```
activity_data$date <- as.Date(activity_data$date)
activity_data$interval <- as.factor(activity_data$interval)
```
activity_data has the following variables of the following classes:
```
names(activity_data)
lapply(activity_data, class)
```

### What is mean total number of steps taken per day?
1. Prepare plot by adding all the number of steps each day and removing missing values (NA)
```
library(plyr)
analysis_1_data <- ddply(activity_data[,1:2], .(date), function(set) { sum(set$steps, na.rm = TRUE) })
names(analysis_1_data) <- c("date", "steps")
```
2. Generate Histogram
```
library(ggplot2)
ggplot(data = analysis_1_data) + aes(x = factor(date), y = steps) + geom_histogram(stat = "identity") + labs(x ="Date", y = "Total number of steps") + theme(axis.text.x=element_text(angle = -90, hjust = 0))
```

3. The average and median number of total steps taken per day are:
```
mean(analysis_1_data$steps)
median(analysis_1_data$steps)
```

### What is the average daily activity pattern?
1. Plot is prepared by calculting the mean for the number of steps for each interval and removing any missing values (coded as NA):
```
analysis_2_data <- ddply(activity_data, .(interval), function(set) { mean(set$steps, na.rm = TRUE) })
names(analysis_2_data) <- c("interval", "steps")
```
2. Generate Chart
```
hour_intervals <- c(12, 24, 36, 48, 60, 72, 84, 96, 108, 120, 132, 144, 156, 168, 180, 192, 204, 216, 228, 240, 252, 264, 276)
ggplot(data = analysis_2_data) + aes(x = factor(interval), y = steps, group = 1) + geom_line() + labs(x ="5-minute interval", y = "Average number of steps across all days") + theme(axis.text.x = element_text(size = 0)) + geom_vline(xintercept= hour_intervals, linetype="dotted") + geom_vline(xintercept= 144, colour = "red") + geom_text(x=144, y = 150, label="12 pm", angle = 90)
```
Each vertical dotted line denotes an hour. The red dotted line denotes the 12:00 mark (afternoon).
The interval with the maximum number of steps is caluclated as follows:
```
analysis_2_data[analysis_2_data$steps == max(analysis_2_data$steps), ]$interval
```

### Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs). 
* There are 2304 missing values for the steps variable (coded as NA) in activity_data:
```
nrow(activity_data[is.na(activity_data$steps), ])
```
2. Missing values will be imputed by calculating the average number of steps for that interval across all days.
```
impute_mean <- function(x) {
  replace(x, is.na(x), mean(x, na.rm = TRUE))
}
```
* An order variable, representing the row order, is added to activity_data so that the data frame with the imputed values can be sorted according to the original order:
```
activity_data$order <- 1:nrow(activity_data)
```
3. NA values are imputed by taking the mean of the total steps for that interval across all days. The resulting dataset is stored in imputed_activity_data:
```
imputed_activity_data <- ddply(activity_data, .(interval), transform, steps = impute_mean(steps))
```
* imputed_activity_data is ordered so that it reflects the original order of activity_data:
```
imputed_activity_data <- imputed_activity_data[order(imputed_activity_data$order), ]
```
* imputed_activity_data has the following variables of the following classes:
```
names(activity_data)
lapply(activity_data, class)
```
4. Histogram can be produced for the total number of steps taken each day. First the data for the plot is prepared by summing the number of steps for each day:
```
analysis_3_data <- ddply(imputed_activity_data[,1:2], .(date), function(set) { sum(set$steps, na.rm = TRUE) })
names(analysis_3_data) <- c("date", "steps")
ggplot(data = analysis_3_data) + aes(x = factor(date), y = steps) + geom_histogram(stat = "identity") + labs(x ="Date", y = "Total number of steps (NA imputed)") + theme(axis.text.x=element_text(angle = -90, hjust = 0))
```
* The average and median number of total steps taken per day, with missing values imputed, are:
```
mean(analysis_3_data$steps)
median(analysis_3_data$steps)
```
* The following table compares the values for steps where missing are present and where missing values have been imputed:

|             |          With missing values                            |         Missing values imputed                          |
|:------------|:--------------------------------------------------------|:--------------------------------------------------------|
| Mean        |   r format(mean(analysis_1_data$steps), digits = 2)     |   r format(mean(analysis_3_data$steps), digits = 2)     |
| Median      |   r format(median(analysis_1_data$steps), digits = 2)   |   r format(median(analysis_3_data$steps), digits = 2)   | 

### Are there differences in activity patterns between weekdays and weekends?
1. Created a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.
* To analyse activity patterns between weekdays and weekends, a vector is created with the weekday for each value of date in imputed_activity_data. A factor variable, which_day, is then added to imputed_activity_data indicating whether the date is on a weekday or a weekend
```
which_day <- weekdays(imputed_activity_data$date)
imputed_activity_data$which_day <- ifelse(which_day == "Saturday" | which_day == "Sunday" , c("weekend"), c("weekday"))
imputed_activity_data$which_day <- factor(imputed_activity_data$which_day)
```
* imputed_activity_data now has the following variables of the following classes:
```
names(imputed_activity_data)
lapply(imputed_activity_data, class)
```
* Two time series plots will be produced:
The average total number of steps taken for each interval across weekdays.
The average total number of steps taken for each interval across weekends.
```
analysis_4_data_weekdays <- ddply(imputed_activity_data[imputed_activity_data$which_day == "weekday", ], .(interval), function(set) { mean(set$steps, na.rm = TRUE) })
names(analysis_4_data_weekdays) <- c("interval", "steps")
analysis_4_data_weekends <- ddply(imputed_activity_data[imputed_activity_data$which_day == "weekend", ], .(interval), function(set) { mean(set$steps, na.rm = TRUE) })
names(analysis_4_data_weekends) <- c("interval", "steps")
```
* Chart objects are generated
```
plot_weekdays <- ggplot(data = analysis_4_data_weekdays) + aes(x = factor(interval), y = steps, group = 1) + geom_line() + labs(x ="5-minute interval", y = "Average number of steps across weekdays") + theme(axis.text.x = element_text(size = 0)) + geom_vline(xintercept= hour_intervals, linetype="dotted") + geom_vline(xintercept= 144, colour = "red") + geom_text(x=144, y = 150, label="12 pm", angle = 90)
plot_weekends <- ggplot(data = analysis_4_data_weekends) + aes(x = factor(interval), y = steps, group = 1) + geom_line() + labs(x ="5-minute interval", y = "Average number of steps across weekends") + theme(axis.text.x = element_text(size = 0)) + geom_vline(xintercept= hour_intervals, linetype="dotted") + geom_vline(xintercept= 144, colour = "red") + geom_text(x=144, y = 150, label="12 pm", angle = 90)
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data
```
multiplot <- function(..., plotlist=NULL, cols) {
  require(grid)
  
  # Make a list from the ... arguments and plotlist
  plots <- c(list(...), plotlist)
  
  numPlots = length(plots)
  
  # Make the panel
  plotCols = cols                          # Number of columns of plots
  plotRows = ceiling(numPlots/plotCols) # Number of rows needed, calculated from # of cols
  
  # Set up the page
  grid.newpage()
  pushViewport(viewport(layout = grid.layout(plotRows, plotCols)))
  vplayout <- function(x, y)
    viewport(layout.pos.row = x, layout.pos.col = y)
  
  # Make each plot, in the correct location
  for (i in 1:numPlots) {
    curRow = ceiling(i/plotCols)
    curCol = (i-1) %% plotCols + 1
    print(plots[[i]], vp = vplayout(curRow, curCol ))
  }
  
}
```
Plots are generated using this function:
```
multiplot(plot_weekdays, plot_weekends, cols = 1)
```
