run.analysis.R
================

``` r
knitr::opts_chunk$set(echo = TRUE)
systimebegin <- Sys.time()
library(data.table)
library(stringr)
library(tidyverse)
```

    ## -- Attaching packages ---------------------------------------------------------- tidyverse 1.2.1 --

    ## v ggplot2 2.2.1.9000     v readr   1.1.1     
    ## v tibble  1.3.4          v purrr   0.2.4     
    ## v tidyr   0.7.2          v dplyr   0.7.4     
    ## v ggplot2 2.2.1.9000     v forcats 0.2.0

    ## -- Conflicts ------------------------------------------------------------- tidyverse_conflicts() --
    ## x dplyr::between()   masks data.table::between()
    ## x dplyr::filter()    masks stats::filter()
    ## x dplyr::first()     masks data.table::first()
    ## x dplyr::lag()       masks stats::lag()
    ## x dplyr::last()      masks data.table::last()
    ## x purrr::transpose() masks data.table::transpose()

### Extract features and activity labels

### Extract subject\_test X\_test & y\_test

### Extract subject\_train X\_train & y\_train

``` r
setwd(dir = "C:\\Users\\yousri.hajri\\Documents\\DATA_SCIENCE\\3_GETTING_AND_CLEANING_DATA\\WEEK4\\run_analysis.R")
source("data.extractor.R")
```

### Merge train&test

``` r
subject <- rbind.data.frame(subject_train, subject_test)
X <- rbind.data.frame(X_train, X_test)
y <- rbind.data.frame(y_train, y_test)
```

### Merge y and activity\_labels

### Merge X and features

``` r
y <- merge (x=y, y=activity_labels, by.x = "V1", by.y = "V1", sort=F) 
y <- select (y, V2) %>% rename(activity_labels=V2)

X <- as.data.frame(X)
X <- rbind(as.vector(features$V2), X)
colnames(X)=X[1,]
X <- X[-1,]
X <- as.data.table(X)
```

### Rename column of subject

``` r
subject <- rename (subject, subject=V1)
```

### Fulldata merger

``` r
fulldata <- cbind(subject, y, X)
```

### Create meanfulldata with colnames containing "mean"

### Create stdfulldata with colnames containing "std"

### Merge meanfulldata and stdfulldata in subdata

``` r
v1 <- c()
for (i in 3:length(colnames(fulldata))) {
              if (length(grep("mean", colnames(fulldata[,i,with=FALSE])))==1) {
               v1 <- c(v1,i)      }}
mean_fulldata <- fulldata[,c(1,2,v1),with=FALSE]

v2 <- c()
for (i in 3:length(colnames(fulldata))) {
  if (length(grep("std", colnames(fulldata[,i,with=FALSE])))==1) {
    v2 <- c(v2,i)      }}
std_fulldata <- fulldata[,v2,with=FALSE]

subdata <- cbind (mean_fulldata, std_fulldata)
```

Class the columns of subdata
----------------------------

``` r
collength <- length(names(subdata))

for(i in 3:collength) {
              subdata[,i] <- as.numeric (as.matrix(subdata[,i,with=FALSE])) }

subdata$activity_labels <- as.factor(subdata$activity_labels)
subdata$subject <- as.factor(subdata$subject)

subdata <- as.tibble (subdata)

subdata <- arrange (subdata, subject)

print(subdata, n=5, n_extra = 5)
```

    ## # A tibble: 10,299 x 81
    ##   subject activity_labels `tBodyAcc-mean()-X` `tBodyAcc-mean()-Y`
    ##    <fctr>          <fctr>               <dbl>               <dbl>
    ## 1       1        STANDING           0.2885845         -0.02029417
    ## 2       1        STANDING           0.2784188         -0.01641057
    ## 3       1        STANDING           0.2796531         -0.01946716
    ## 4       1        STANDING           0.2791739         -0.02620065
    ## 5       1        STANDING           0.2766288         -0.01656965
    ## # ... with 1.029e+04 more rows, and 77 more variables:
    ## #   `tBodyAcc-mean()-Z` <dbl>, `tGravityAcc-mean()-X` <dbl>,
    ## #   `tGravityAcc-mean()-Y` <dbl>, `tGravityAcc-mean()-Z` <dbl>,
    ## #   `tBodyAccJerk-mean()-X` <dbl>, ...

Summarise all subject/activity\_labels by mean function
-------------------------------------------------------

``` r
final_data <- subdata %>%
  group_by(subject, activity_labels) %>%
  summarise_all (funs(mean))

print (final_data, n=5, n_extra = 5)
```

    ## # A tibble: 180 x 81
    ## # Groups:   subject [?]
    ##   subject    activity_labels `tBodyAcc-mean()-X` `tBodyAcc-mean()-Y`
    ##    <fctr>             <fctr>               <dbl>               <dbl>
    ## 1       1             LAYING           0.2215982        -0.040513953
    ## 2       1            SITTING           0.2612376        -0.001308288
    ## 3       1           STANDING           0.2789176        -0.016137590
    ## 4       1            WALKING           0.2773308        -0.017383819
    ## 5       1 WALKING_DOWNSTAIRS           0.2891883        -0.009918505
    ## # ... with 175 more rows, and 77 more variables:
    ## #   `tBodyAcc-mean()-Z` <dbl>, `tGravityAcc-mean()-X` <dbl>,
    ## #   `tGravityAcc-mean()-Y` <dbl>, `tGravityAcc-mean()-Z` <dbl>,
    ## #   `tBodyAccJerk-mean()-X` <dbl>, ...

### Remove old data

### Performance

### END

``` r
remove (subject, X, y)
remove (subject_train, subject_test,
        X_train, X_test,
        y_train, y_test)
remove (activity_labels, features)
remove (fulldata, mean_fulldata, std_fulldata)

systimeend <- Sys.time()
runtime <- systimeend - systimebegin
print(runtime)
```

    ## Time difference of 49.49598 secs
