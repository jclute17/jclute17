Bellabeat Case Study Using R
================
Jay Clute
October 11th, 2023

### About This Study

This is my first case study upon completing Google’s Data Analytics
certification course. The purpose of this case study is to get hands-on
experience working with a data set, processing and analyzing it,
creating data visualizations, and offering possible solutions to the
stakeholders. The entirety of this study has been completed through R
and the tidyverse package, from importing the data set, to manipulating
sheets, to creating the data visualizations.

#### About The Company

Bellabeat is a high-tech manufacturer of health-focused products for
women which has the potential to become a large player in the global
smart device market. They believe that analyzing smart device fitness
data could help unlock new growth opportunities for the company. I have
been asked to analyze smart device data to gain insight into how
consumers are using their smart devices. I have also been asked to give
high-level recommendations for Bellabeat’s marketing strategy.

#### Questions to Focus On

- What are some trends in smart device usage?
- How could these trends apply to Bellabeat consumers?
- How could these trends help influence Bellabeat marketing strategy?

#### Business Task

**Analyze trends in smart device usage in order to identify potential
growth opportunities and recommendations for the Bellabeat marketing
team.**

##### Internal Links

Feel free to skip ahead to any sections that interest you!  
1. [Understanding the Data](#understanding-the-data)  
2. [Using R](#using-r)  
3. [Load Tidyverse](#load-tidyverse)  
4. [Import Datasets](#import-datasets)  
5. [Exploring the Data](#exploring-the-data)  
6. [Merging Data](#merging-data)  
7. [Visualizations](#visualizations)  
8. [Diving Deeper](#diving-deeper)  
9. [Recommendations](#recommendations)  
10. [References](#references)

$$\\[.2in]$$

### Understanding the Data

Before diving into R, I wanted to fully understand the data that I was
working with. I was given 18 separate data sets, so I needed to know
what was going on with each one. I did this by looking into each .csv
file and recording the variables and how they were recorded. Once I had
them written out, I looked to see what .csv files could be joined with
others to show insights. This also allowed me to come up with certain
hypotheses to test, as well as assumptions I had to make due to a lack
of proper column names or notes/comments.

Assumption(s) Worth Noting

- “MET” stands for “Metabolic Equivalent of Task” according to an
  article from the *Harvard School of Public Health*.

Hypotheses

- Does total steps correlate with calories burned?
- Does high MET correlate with high calories burned?
- Are participants wasting too much time in bed but not sleeping?
  i.e. scrolling on social media

### Using R

###### Load Tidyverse

First, we need to install and load the necessary packages that will be
needed to work with the data sets.

``` r
  library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.3     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.0
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.2     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

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

    ##           Id              SleepDay TotalSleepRecords TotalMinutesAsleep
    ## 1 1503960366 4/12/2016 12:00:00 AM                 1                327
    ## 2 1503960366 4/13/2016 12:00:00 AM                 2                384
    ## 3 1503960366 4/15/2016 12:00:00 AM                 1                412
    ## 4 1503960366 4/16/2016 12:00:00 AM                 2                340
    ## 5 1503960366 4/17/2016 12:00:00 AM                 1                700
    ## 6 1503960366 4/19/2016 12:00:00 AM                 1                304
    ##   TotalTimeInBed
    ## 1            346
    ## 2            407
    ## 3            442
    ## 4            367
    ## 5            712
    ## 6            320

While checking, I see that time and date are in the same column, which
will make it difficult for analyzing. So, I split up date and time into
their own separate columns, while still keeping the original.

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

I can now see that date and time are in their own separate columns, so I
can move on to exploring the data.

$$\\[.2in]$$

##### Exploring the Data

First, I’d like to see how many participants are in each data set to get
an idea of sample size.

``` r
  n_distinct(activity$Id)
```

    ## [1] 33

``` r
  n_distinct(calories$Id)
```

    ## [1] 33

``` r
  n_distinct(intensities$Id)
```

    ## [1] 33

``` r
  n_distinct(sleep$Id)
```

    ## [1] 24

``` r
  n_distinct(weight$Id)
```

    ## [1] 8

``` r
  n_distinct(mets$Id)
```

    ## [1] 33

``` r
  n_distinct(minsCalories$Id)
```

    ## [1] 33

We see that all data sets have 33 participants besides for “sleep” which
has 24 and “weights” which only has 8. 8 is not a big enough sample size
to be confident in giving any recommendations, so this data set will not
be used. Next, I would like to get a summary of the data to see if there
are any interesting averages.

``` r
  # activity
  activity %>% 
    select(TotalSteps,TotalDistance,SedentaryMinutes, Calories) %>% 
    summary()
```

    ##    TotalSteps    TotalDistance    SedentaryMinutes    Calories   
    ##  Min.   :    0   Min.   : 0.000   Min.   :   0.0   Min.   :   0  
    ##  1st Qu.: 3790   1st Qu.: 2.620   1st Qu.: 729.8   1st Qu.:1828  
    ##  Median : 7406   Median : 5.245   Median :1057.5   Median :2134  
    ##  Mean   : 7638   Mean   : 5.490   Mean   : 991.2   Mean   :2304  
    ##  3rd Qu.:10727   3rd Qu.: 7.713   3rd Qu.:1229.5   3rd Qu.:2793  
    ##  Max.   :36019   Max.   :28.030   Max.   :1440.0   Max.   :4900

``` r
  # activity, see number of active mins per category
  activity %>% 
    select(VeryActiveMinutes, FairlyActiveMinutes, LightlyActiveMinutes) %>% 
    summary()
```

    ##  VeryActiveMinutes FairlyActiveMinutes LightlyActiveMinutes
    ##  Min.   :  0.00    Min.   :  0.00      Min.   :  0.0       
    ##  1st Qu.:  0.00    1st Qu.:  0.00      1st Qu.:127.0       
    ##  Median :  4.00    Median :  6.00      Median :199.0       
    ##  Mean   : 21.16    Mean   : 13.56      Mean   :192.8       
    ##  3rd Qu.: 32.00    3rd Qu.: 19.00      3rd Qu.:264.0       
    ##  Max.   :210.00    Max.   :143.00      Max.   :518.0

``` r
  # calories
  calories %>% 
    select(Calories) %>% 
    summary()
```

    ##     Calories     
    ##  Min.   : 42.00  
    ##  1st Qu.: 63.00  
    ##  Median : 83.00  
    ##  Mean   : 97.39  
    ##  3rd Qu.:108.00  
    ##  Max.   :948.00

``` r
  # sleep
  sleep %>% 
    select(TotalSleepRecords, TotalMinutesAsleep, TotalTimeInBed) %>% 
    summary()
```

    ##  TotalSleepRecords TotalMinutesAsleep TotalTimeInBed 
    ##  Min.   :1.000     Min.   : 58.0      Min.   : 61.0  
    ##  1st Qu.:1.000     1st Qu.:361.0      1st Qu.:403.0  
    ##  Median :1.000     Median :433.0      Median :463.0  
    ##  Mean   :1.119     Mean   :419.5      Mean   :458.6  
    ##  3rd Qu.:1.000     3rd Qu.:490.0      3rd Qu.:526.0  
    ##  Max.   :3.000     Max.   :796.0      Max.   :961.0

``` r
  # weight
  weight %>% 
    select(WeightPounds, BMI) %>% 
    summary()
```

    ##   WeightPounds        BMI       
    ##  Min.   :116.0   Min.   :21.45  
    ##  1st Qu.:135.4   1st Qu.:23.96  
    ##  Median :137.8   Median :24.39  
    ##  Mean   :158.8   Mean   :25.19  
    ##  3rd Qu.:187.5   3rd Qu.:25.56  
    ##  Max.   :294.3   Max.   :47.54

Interesting notes from the summaries:

- 991.2mins or 16.52 hours is the average sedentary time per
  participant. Bellabeat could try to encourage users to decrease time
  spent sedentary
- Average steps per day are 7,638 which is a little under CDC
  recommendations. They say that 8,000 steps per day is associated with
  a 51% lower risk for all-cause mortality while 12,000 steps per day is
  associated with 65% lower risk for all-cause mortality. Bellabeat can
  notify users to get up and go for a walk to try to hit the 8,000 to
  12,000 step range.
- An overwhelming majority of participants are only lightly active.
- On average, each participant lies awake in bed for 30mins a day. I
  wonder if there are some participants who lie awake for much longer
  than this. I will test this by creating a data viz on time in bed vs
  time asleep.
- About half of the participants fall in the categories of overweight or
  obese according to the CDC Healthy Weight Range. However, since the
  sample size is so small for this data set, I will not give
  recommendations based on this finding.

$$\\[.2in]$$

##### Merging Data

The activity data set holds the most data, but I would like to add in
the sleep data as well. I am going to join these two sets and call it
master_sheet. I would also like to merge mets and minsCalories to be
able to compare the data to see if there is a correlation between METs
and calories burnt

``` r
  master_sheet <- merge(sleep, activity, by = c('Id', 'date'))
  metCals <- merge(mets, minsCalories, by = c('Id', 'ActivityMinute'))
  head(master_sheet)
```

    ##           Id     date   SleepDay TotalSleepRecords TotalMinutesAsleep
    ## 1 1503960366 04/12/16 2016-04-12                 1                327
    ## 2 1503960366 04/13/16 2016-04-13                 2                384
    ## 3 1503960366 04/15/16 2016-04-15                 1                412
    ## 4 1503960366 04/16/16 2016-04-16                 2                340
    ## 5 1503960366 04/17/16 2016-04-17                 1                700
    ## 6 1503960366 04/19/16 2016-04-19                 1                304
    ##   TotalTimeInBed ActivityDate TotalSteps TotalDistance TrackerDistance
    ## 1            346   2016-04-12      13162          8.50            8.50
    ## 2            407   2016-04-13      10735          6.97            6.97
    ## 3            442   2016-04-15       9762          6.28            6.28
    ## 4            367   2016-04-16      12669          8.16            8.16
    ## 5            712   2016-04-17       9705          6.48            6.48
    ## 6            320   2016-04-19      15506          9.88            9.88
    ##   LoggedActivitiesDistance VeryActiveDistance ModeratelyActiveDistance
    ## 1                        0               1.88                     0.55
    ## 2                        0               1.57                     0.69
    ## 3                        0               2.14                     1.26
    ## 4                        0               2.71                     0.41
    ## 5                        0               3.19                     0.78
    ## 6                        0               3.53                     1.32
    ##   LightActiveDistance SedentaryActiveDistance VeryActiveMinutes
    ## 1                6.06                       0                25
    ## 2                4.71                       0                21
    ## 3                2.83                       0                29
    ## 4                5.04                       0                36
    ## 5                2.51                       0                38
    ## 6                5.03                       0                50
    ##   FairlyActiveMinutes LightlyActiveMinutes SedentaryMinutes Calories
    ## 1                  13                  328              728     1985
    ## 2                  19                  217              776     1797
    ## 3                  34                  209              726     1745
    ## 4                  10                  221              773     1863
    ## 5                  20                  164              539     1728
    ## 6                  31                  264              775     2035

``` r
  head(metCals)
```

    ##           Id      ActivityMinute METs   time.x   date.x Calories   time.y
    ## 1 1503960366 2016-04-12 00:00:00   10 00:00:00 04/12/16   0.7865 00:00:00
    ## 2 1503960366 2016-04-12 00:01:00   10 00:01:00 04/12/16   0.7865 00:01:00
    ## 3 1503960366 2016-04-12 00:02:00   10 00:02:00 04/12/16   0.7865 00:02:00
    ## 4 1503960366 2016-04-12 00:03:00   10 00:03:00 04/12/16   0.7865 00:03:00
    ## 5 1503960366 2016-04-12 00:04:00   10 00:04:00 04/12/16   0.7865 00:04:00
    ## 6 1503960366 2016-04-12 00:05:00   12 00:05:00 04/12/16   0.9438 00:05:00
    ##     date.y
    ## 1 04/12/16
    ## 2 04/12/16
    ## 3 04/12/16
    ## 4 04/12/16
    ## 5 04/12/16
    ## 6 04/12/16

The data merged properly, now I can move on to creating data
visualizations to help the stakeholders see trends in the data.

$$\\[.2in]$$

##### Visualizations

I will first start with my first hypothesis: Does total steps correlate
to calories burnt?

``` r
  ggplot(data=activity, aes(x=TotalSteps, y=Calories)) + geom_point() + geom_smooth() + labs(title="Total Steps vs. Calories Burnt")
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

![](BellabeatCaseStudy_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

We see positive correlation here between total steps vs calories burnt;
the more steps we take, the more calories we burn. Now, let’s move on to
MET’s vs Calories.

``` r
  ggplot(data=metCals, aes(x=METs, y=Calories)) + geom_point() + geom_smooth() + labs(title="METs vs. Calories")
```

    ## `geom_smooth()` using method = 'gam' and formula = 'y ~ s(x, bs = "cs")'

![](BellabeatCaseStudy_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

There is a clear correlation here with METs (Metabolic Equivalent of
Task) vs. Calories. However, it seems that the points are in lines, and
it is quite possible that each line is a participant. If that is the
case, then MET is most likely calculated using the calories burned, so
we can not use this relationship/visualization for insights. From here
we can move on to minutes asleep vs total time in bed.

``` r
  ggplot(data=sleep, aes(x=TotalMinutesAsleep, y=TotalTimeInBed)) + geom_point() + labs(title="Total Minutes Asleep vs Total Time in Bed")
```

![](BellabeatCaseStudy_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

We see that some participants spend a lot of time in bed before actually
going to sleep. Bellabeat could give notifications to their users if
they spend too much time on their phone in bed before going to sleep.

$$\\[.1in]$$

###### Diving Deeper

I want to look at intensities, but first, I’ll need to make a new data
set that uses the average intensities grouped by time. Then, I can plot
the average intensities based on time.

``` r
  int_new <- intensities %>% group_by(time) %>% drop_na() %>% summarise(mean_total_int = mean(TotalIntensity))
  ggplot(data=int_new, aes(x=time, y=mean_total_int)) + geom_histogram(stat = "identity", fill = 'darkblue') + theme(axis.text.x = element_text(angle = 90)) + labs(title="Average Total Intensity vs. Time")
```

    ## Warning in geom_histogram(stat = "identity", fill = "darkblue"): Ignoring
    ## unknown parameters: `binwidth`, `bins`, and `pad`

![](BellabeatCaseStudy_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

From this chart, we see that people are more active from 5am to 11pm.
Peak activity seems to occur between 5pm and 7pm. One reason for this is
because that is when most people get out of work so they may go to the
gym after work or take their dog for a walk once they get off work.
Bellabeat could give reminders to people to go for a walk after work or
even try to encourage users to get up and be active at other times of
the day such as before work or during lunch. After finding this out, I
want to see if there is a correlation between time spent asleep and
sedentary minutes.

``` r
  ggplot(data=master_sheet, aes(x=SedentaryMinutes, y=TotalMinutesAsleep)) + geom_point(color = 'darkblue') + geom_smooth() + labs(title="Sedentary Minutes vs. Minutes Asleep")
```

    ## `geom_smooth()` using method = 'loess' and formula = 'y ~ x'

![](BellabeatCaseStudy_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

Here, we can see a negative correlation between sedentary minutes and
sleep time; the more sedentary someone is, the worse sleep they get.
Bellabeat can recommend users to reduce their sedentary time if they
want to improve their sleep.

$$\\[.1in]$$

#### Recommendations

There are a few insights that have been found through the research of
these data sets.

- Participants are not walking as much as the CDC recommends. The CDC
  says that taking 8,000 steps per day is associated with a 51% lower
  risk for all-cause mortality and percentage increases to 65% lower
  risk for those who walk over 12,000 steps a day. Bellabeat could give
  notifications when it can tell that someone is not going to make it to
  the 8,000+ step range.
- If users are stuck scrolling on social media while in bed, Bellabeat
  can give sleep notifications to remind their users to put their phone
  away and get some sleep.  
- Bellabeat can encourage users to be more active, as it was found that
  those who are more sedentary tend to have worse sleep.

##### References

**Thank you** for taking the time to read over my Bellabeat Case Study!
This would not have been possible if it weren’t for the guiding work of
Anastasia Chebotina and her previous [case
study](https://www.kaggle.com/code/chebotinaa/bellabeat-case-study-with-r)
that helped me and gave me the confidence to take this research a step
further!
