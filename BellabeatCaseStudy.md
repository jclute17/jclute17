---
title: "Bellabeat Case Study Using R"
author: "Jay Clute"
date: "October 11th, 2023"
output: git_document
---

### About This Study

This is my first case study upon completing Google's Data Analytics certification course. The purpose of this case study is to get hands-on experience working with a data set, processing and analyzing it, creating data visualizations, and offering possible solutions to the stakeholders. The entirety of this study has been completed through R and the tidyverse package, from importing the data set, to manipulating sheets, to creating the data visualizations.

#### About The Company

Bellabeat is a high-tech manufacturer of health-focused products for women which has the potential to become a large player in the global smart device market. They believe that analyzing smart device fitness data could help unlock new growth opportunities for the company. I have been asked to analyze smart device data to gain insight into how consumers are using their smart devices. I have also been asked to give high-level recommendations for Bellabeat's marketing strategy.

#### Questions to Focus On

-   What are some trends in smart device usage?
-   How could these trends apply to Bellabeat consumers?
-   How could these trends help influence Bellabeat marketing strategy?

#### Business Task

**Analyze trends in smart device usage in order to identify potential growth opportunities and recommendations for the Bellabeat marketing team.**

$$\\[.2in]$$

### Understanding the Data

Before diving into R, I wanted to fully understand the data that I was working with. I was given 18 separate data sets, so I needed to know what was going on with each one. I did this by looking into each .csv file and recording the variables and how they were recorded. Once I had them written out, I looked to see what .csv files could be joined with others to show insights. This also allowed me to come up with certain hypotheses to test, as well as assumptions I had to make due to a lack of proper column names or notes/comments.

Assumption(s) Worth Noting

-   "MET" stands for "Metabolic Equivalent of Task" according to an article from the *Harvard School of Public Health*.

Hypotheses

-   Does total steps correlate with calories burned?
-   Does high MET correlate with high calories burned?
-   Are participants wasting too much time in bed but not sleeping? i.e. scrolling on social media

### Using R

###### Load and Install Packages

First, we need to install and load the necessary packages that will be needed to work with the data sets.

``` r
  install.packages("tidyverse")
  install.packages("lubridate")
  install.packages("dplyr")
  install.packages("ggplot2")
  install.packages("tidyr")
  
  library(tidyverse)
  library(lubridate)
  library(dplyr)
  library(ggplot2)
  library(tidyr)
```

$$\\[.1in]$$

##### Import Datasets

The given data for this project is the FitBit Tracker Data

``` r
  sleep <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/sleepDay_merged.csv")
  activity <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/dailyActivity_merged.csv")
  calories <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/hourlyCalories_merged.csv")
  intensities <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/hourlyIntensities_merged.csv")
  weight <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/weightLogInfo_merged.csv")
  mets <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/minuteMETsNarrow_merged.csv")
  minsCalories <- read.csv("/Users/jayclute/Downloads/Fitabase Data 4.12.16-5.12.16/minuteCaloriesNarrow_merged.csv")
```

I then checked each data set to ensure it was imported properly

``` r
head(sleep)
```

While checking, I see that time and date are in the same column, which will make it difficult for analyzing. So, I split up date and time into their own separate columns, while still keeping the original.

``` r
  # intensities 
  intensities$ActivityHour=as.POSIXct(intensities$ActivityHour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()) 
  intensities$time <- format(intensities$ActivityHour, format = "%H:%M:%S") 
  intensities$date <- format(intensities$ActivityHour, format = "%m/%d/%y")

  # calories 
  calories$ActivityHour=as.POSIXct(calories$ActivityHour, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()) 
  calories$time <- format(calories$ActivityHour, format = "%H:%M:%S") 
  calories$date <- format(calories$ActivityHour, format = "%m/%d/%y")

  # activity 
  activity$ActivityDate=as.POSIXct(activity$ActivityDate, format="%m/%d/%Y", tz=Sys.timezone()) 
  activity$date <- format(activity$ActivityDate, format = "%m/%d/%y")

  # sleep 
  sleep$SleepDay=as.POSIXct(sleep$SleepDay, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()) 
  sleep$date <- format(sleep$SleepDay, format = "%m/%d/%y")

  # mets 
  mets$ActivityMinute=as.POSIXct(mets$ActivityMinute, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()) 
  mets$time <- format(mets$ActivityMinute, format = "%H:%M:%S") 
  mets$date <- format(mets$ActivityMinute, format = "%m/%d/%y")

  # minsCalories 
  minsCalories$ActivityMinute=as.POSIXct(minsCalories$ActivityMinute, format="%m/%d/%Y %I:%M:%S %p", tz=Sys.timezone()) 
  minsCalories$time <- format(minsCalories$ActivityMinute, format = "%H:%M:%S") 
  minsCalories$date <- format(minsCalories$ActivityMinute, format = "%m/%d/%y")
```

I can now see that date and time are in their own separate columns, so I can move on to exploring the data.

$$\\[.2in]$$

##### Exploring the Data

First, I'd like to see how many participants are in each data set to get an idea of sample size.

``` r
  n_distinct(activity$Id)
  n_distinct(calories$Id)
  n_distinct(intensities$Id)
  n_distinct(sleep$Id)
  n_distinct(weight$Id)
  n_distinct(mets$Id)
  n_distinct(minsCalories$Id)
```

We see that all data sets have 33 participants besides for "sleep" which has 24 and "weights" which only has 8. 8 is not a big enough sample size to be confident in giving any recommendations, so this data set will not be used. Next, I would like to get a summary of the data to see if there are any interesting averages.

``` r
  # activity
  activity %>% 
    select(TotalSteps,TotalDistance,SedentaryMinutes, Calories) %>% 
    summary()
    
  # activity, see number of active mins per category
  activity %>% 
    select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes) %>% 
    summary()
    
  # calories
  calories %>% 
    select(Calories) %>% 
    summary()
  
  # sleep
  sleep %>% 
    select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>% 
    summary()
  
  # weight
  weight %>% 
    select(WeightPounds, BMI) %>% 
    summary()
```

Interesting notes from the summaries:

-   991.2mins or 16.52 hours is the average sedentary time per participant. Bellabeat could try to encourage users to decrease time spent sedentary
-   Average steps per day are 7,638 which is a little under CDC recommendations. They say that 8,000 steps per day is associated with a 51% lower risk for all-cause mortality while 12,000 steps per day is associated with 65% lower risk for all-cause mortality. Bellabeat can notify users to get up and go for a walk to try to hit the 8,000 to 12,000 step range.
-   An overwhelming majority of participants are only lightly active.
-   On average, each participant lies awake in bed for 30mins a day. I wonder if there are some participants who lie awake for much longer than this. I will test this by creating a data viz on time in bed vs time asleep.
-   About half of the participants fall in the categories of overweight or obese according to the CDC Healthy Weight Range. However, since the sample size is so small for this data set, I will not give recommendations based on this finding.

$$\\[.2in]$$

##### Merging Data

The activity data set holds the most data, but I would like to add in the sleep data as well. I am going to join these two sets and call it master_sheet. I would also like to merge mets and minsCalories to be able to compare the data to see if there is a correlation between METs and calories burnt

``` r
  master_sheet <- merge(sleep, activity, by = c('Id', 'date'))
  metCals <- merge(mets, minsCalories, by = c(‘Id’, ‘ActivityMinute’))
  head(master_sheet)
  head(metCals)
```

The data merged properly, now I can move on to creating data visualizations to help the stakeholders see trends in the data.

$$\\[.2in]$$

##### Visualizations

I will first start with my first hypothesis: Does total steps correlate to calories burnt?

``` r
  ggplot(data=activity, aes(x=TotalSteps, y=Calories)) + geom_point() + geom_smooth() + labs(title="Total Steps vs. Calories Burnt")
```

We see positive correlation here between total steps vs calories burnt; the more steps we take, the more calories we burn.
