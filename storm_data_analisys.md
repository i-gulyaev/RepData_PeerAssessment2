---
title: 'Reproducible Research: Peer Assessment 2'
output:
  pdf_document: default
  html_document:
    keep_md: yes
---

# What weather events influence to economics and public health 
The aim of this report is to explor U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database and answer to the following questions
1. Across the United States, which types of events (as indicated in the EVTYPE variable) are most harmful with respect to population health?
2. Across the United States, which types of events have the greatest economic consequences?
During data analisys we identified weather events with the greatest consequences w.r.t to public health and economy based on number of victims and economic loss.

## Data Processing
U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database
which is explored in this report is available by the link [Storm Data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2) [47Mb].
This database tracks characteristics of major storms and weather events in
the United States, including when and where they occur, as well as estimates of
any fatalities, injuries, and property damage.


### Reading the data
Storm database is a CSV file compressed by bzip2 algorithm.
We read it as is and explore its content.

```r
storm.data <- read.csv("stormdata.csv.bz2", na.strings="")
```

Show first few rows of the data file.

```r
dim(storm.data)
```

```
## [1] 902297     37
```

```r
head(storm.data)
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
## 2 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
## 3 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
## 4 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
## 5 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
## 6 TORNADO         0    <NA>       <NA>     <NA>     <NA>          0
##   COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1         NA         0    <NA>       <NA>   14.0   100 3   0          0
## 2         NA         0    <NA>       <NA>    2.0   150 2   0          0
## 3         NA         0    <NA>       <NA>    0.1   123 2   0          0
## 4         NA         0    <NA>       <NA>    0.0   100 2   0          0
## 5         NA         0    <NA>       <NA>    0.0   150 2   0          0
## 6         NA         0    <NA>       <NA>    1.5   177 2   0          0
##   INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP  WFO STATEOFFIC ZONENAMES
## 1       15    25.0          K       0       <NA> <NA>       <NA>      <NA>
## 2        0     2.5          K       0       <NA> <NA>       <NA>      <NA>
## 3        2    25.0          K       0       <NA> <NA>       <NA>      <NA>
## 4        2     2.5          K       0       <NA> <NA>       <NA>      <NA>
## 5        2     2.5          K       0       <NA> <NA>       <NA>      <NA>
## 6        6     2.5          K       0       <NA> <NA>       <NA>      <NA>
##   LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1     3040      8812       3051       8806    <NA>      1
## 2     3042      8755          0          0    <NA>      2
## 3     3340      8742          0          0    <NA>      3
## 4     3458      8626          0          0    <NA>      4
## 5     3412      8642          0          0    <NA>      5
## 6     3450      8748          0          0    <NA>      6
```

### Preparing data for analisys
From the storm databes we select only those columns which help to determine
economic and health consequeties of weather events. Examining the [Storm Data Documentation](https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf)
and the actual data (see data content above) it's seen that *FATALITIES* and
 *INJURIES* columns correspond to public health, columns *PROPDMG*, *PROPDMGEXP*,
 *CROPDMG* and *CROPDMGEXP* are related to econmic consequences of the weather events.


```r
library(dplyr)
storm.cons <- tbl_df(storm.data) %>% select(STATE__, BGN_DATE, STATE, EVTYPE, FATALITIES,INJURIES, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP)

head(storm.cons)
```

```
## Source: local data frame [6 x 10]
## 
##   STATE__           BGN_DATE STATE  EVTYPE FATALITIES INJURIES PROPDMG
## 1       1  4/18/1950 0:00:00    AL TORNADO          0       15    25.0
## 2       1  4/18/1950 0:00:00    AL TORNADO          0        0     2.5
## 3       1  2/20/1951 0:00:00    AL TORNADO          0        2    25.0
## 4       1   6/8/1951 0:00:00    AL TORNADO          0        2     2.5
## 5       1 11/15/1951 0:00:00    AL TORNADO          0        2     2.5
## 6       1 11/15/1951 0:00:00    AL TORNADO          0        6     2.5
## Variables not shown: PROPDMGEXP (fctr), CROPDMG (dbl), CROPDMGEXP (fctr)
```

### Converting units
Property and crop damages are presented in dollars as numeric value and its
 magnitude ("1.55 B" - 1.55 billion dollars)
Available magnitude levels are *K* for thousands, *M* for millions, *B* for billions.
As we see in actual data has more than these three levels:

```r
unique(storm.cons$PROPDMGEXP)
```

```
##  [1] K    M    <NA> B    m    +    0    5    6    ?    4    2    3    h   
## [15] 7    H    -    1    8   
## Levels: - ? + 0 1 2 3 4 5 6 7 8 B h H K m M
```

```r
unique(storm.cons$CROPDMGEXP)
```

```
## [1] <NA> M    K    m    B    ?    0    k    2   
## Levels: ? 0 2 B k K m M
```

From the Storm Data Documentation meaning of the rest of the levels is unclear.
So all unknown magnitued levels will be ignored. Only numeric part will be considered.


```r
convert.magnitude <- function(value) {
    if (is.na(value)) {
        1
    } else if (value == "K" | value == "k") {
        1000
    } else if (value == "M" | value == "m") {
        1000000
    } else if (value == "B" | value == "b") {
        1000000000
    } else {
        1
    }
}

storm.cons$PROPDMGEXP <- sapply(storm.cons$PROPDMGEXP, convert.magnitude)
storm.cons$CROPDMGEXP <- sapply(storm.cons$CROPDMGEXP, convert.magnitude)
storm.cons <- storm.cons %>% mutate(PROPDMG=PROPDMG*PROPDMGEXP,
CROPDMG=CROPDMG*CROPDMGEXP) %>% select(-PROPDMGEXP, -CROPDMGEXP)
```

### Imputing missing values
There is no missing values in columns that are needed for analisys.

```r
sum(is.na(storm.cons$PROPDMG))
```

```
## [1] 0
```

```r
sum(is.na(storm.cons$CROPDMG))
```

```
## [1] 0
```

```r
sum(is.na(storm.cons$FATALITIES))
```

```
## [1] 0
```

```r
sum(is.na(storm.cons$INJURIES))
```

```
## [1] 0
```

## Results

### Most harmful weather events with respect to population health acrross the United States
Consider number of victims as a weather event effect on population health.
Number of victims is a sum of fatalities and injures for each event type.

```r
ph <- storm.cons %>%
    mutate(VICTIMS=FATALITIES+INJURIES) %>%
    group_by(EVTYPE) %>%
    summarise(VICTIMS=sum(VICTIMS)) %>%
    arrange(desc(VICTIMS))
```

Top 10 most harmfull weather events with respect to population health

```r
head(ph,n=10)
```

```
## Source: local data frame [10 x 2]
## 
##               EVTYPE VICTIMS
## 1            TORNADO   96979
## 2     EXCESSIVE HEAT    8428
## 3          TSTM WIND    7461
## 4              FLOOD    7259
## 5          LIGHTNING    6046
## 6               HEAT    3037
## 7        FLASH FLOOD    2755
## 8          ICE STORM    2064
## 9  THUNDERSTORM WIND    1621
## 10      WINTER STORM    1527
```
As seen from above table the weather event most harmfult w.r.t public health is *tornado*

Plot below shows victims of tornado through the whole history of observations (1950-2011).
Red line on the plot shows the median of victims.

```r
library(lubridate)
tornado <- storm.cons %>%
    mutate(YEAR=year(mdy_hms(BGN_DATE))) %>%
    filter(EVTYPE=="TORNADO") %>%
    mutate(VICTIMS= FATALITIES + INJURIES) %>%
    select(YEAR, VICTIMS) %>%
    group_by(YEAR) %>%
    summarise(VICTIMS=sum(VICTIMS))

plot(tornado, main="Victims of tornado across the United States")
abline(h=median(tornado$VICTIMS), col="red")
```

![plot of chunk victims_of_tornado](figure/victims_of_tornado-1.png) 

### Weather events with the greatest economic consequneses across the United States
Economic consequenses caused by weather events are of two types property damages and crop demages.
Sum of these two factors is a measure of weather event's efffect to economics.


```r
economic.loss <- storm.cons %>%
    mutate(ECONOMIC.LOSS=(PROPDMG+CROPDMG)/1000000) %>%
    group_by(EVTYPE) %>%
    summarise(ECONOMIC.LOSS=sum(ECONOMIC.LOSS)) %>%
    arrange(desc(ECONOMIC.LOSS))
```

Below is the top 10 weather event type with the greatest economic consequenses. Values are in millions of dollars.

```r
head(economic.loss, n=10)
```

```
## Source: local data frame [10 x 2]
## 
##               EVTYPE ECONOMIC.LOSS
## 1              FLOOD    150319.678
## 2  HURRICANE/TYPHOON     71913.713
## 3            TORNADO     57352.114
## 4        STORM SURGE     43323.541
## 5               HAIL     18758.222
## 6        FLASH FLOOD     17562.129
## 7            DROUGHT     15018.672
## 8          HURRICANE     14610.229
## 9        RIVER FLOOD     10148.405
## 10         ICE STORM      8967.041
```
Let's see how weather events affects the economic of the United States fot the past ten years


```r
past10 <- storm.cons %>%
    mutate(YEAR=year(mdy_hms(BGN_DATE))) %>%
    filter(YEAR > 1990) %>%
    mutate(ECONOMIC.LOSS=(PROPDMG+CROPDMG)/1000000) %>%
    select(YEAR, ECONOMIC.LOSS) %>%
    group_by(YEAR) %>%
    summarise(ECONOMIC.LOSS=sum(ECONOMIC.LOSS))

plot(past10, main="Economic loss for various weather events for past 10 years", ylab="Economic loss (millions of dollars)")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 
