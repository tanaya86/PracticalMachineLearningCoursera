
## Title: "Practical Machine Learning - Selection of the Predictive Model"


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## R Markdown

This is an R Markdown document. Markdown is a simple formatting syntax for authoring HTML, PDF, and MS Word documents. For more details on using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that includes both content as well as the output of any embedded R code chunks within the document. You can embed an R code chunk like this:

## Overview


Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, our goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. We were asked to perform barbell lifts correctly and incorrectly in 5 different ways. The Information is available from the website here: http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har.


## Summary


This human activity recognition research has traditionally focused on discriminating between different activities, i.e. to predict "which" activity was performed at a specific point in time (like with the Daily Living Activities dataset above). The approach we propose for the Weight Lifting Exercises dataset is to investigate "how (well)" an activity was performed by the wearer. The "how (well)" investigation has only received little attention so far, even though it potentially provides useful information for a large variety of applications,such as sports training.

In this work we first define quality of execution and investigate three aspects that pertain to qualitative activity recognition: the problem of specifying correct execution, the automatic and robust detection of execution mistakes, and how to provide feedback on the quality of execution to the user. We tried out an on-body sensing approach (dataset here), but also an "ambient sensing approach".

Six young health participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).

Class A corresponds to the specified execution of the exercise, while the other 4 classes correspond to common mistakes. Participants were supervised by an experienced weight lifter to make sure the execution complied to the manner they were supposed to simulate. The exercises were performed by six male participants aged between 20-28 years, with little weight lifting experience. We made sure that all participants could easily simulate the mistakes in a safe and controlled manner by using a relatively light dumbbell (1.25kg).


Data

The training data for this project are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here:

https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv



# Data Processing

```{r}
# Libraries
library(knitr)
library(dplyr)
library(rpart)
library(rpart.plot)
library(rattle)
library(randomForest)
library(corrplot)
library(caret)

library(RColorBrewer)
library(gbm)

# Set seed to create reproducibility
set.seed(12345)
```

Attaching package: 'dplyr'

The following objects are masked from 'package:stats':

filter, lag

The following objects are masked from 'package:base':

intersect, setdiff, setequal, union

Loading required package: tibble

Loading required package: bitops

Rattle: A free graphical interface for data science with R.

Version 5.4.0 Copyright (c) 2006-2020 Togaware Pty Ltd.

Type 'rattle()' to shake, rattle, and roll your data.

randomForest 4.6-14

Type rfNews() to see new features/changes/bug fixes.

Attaching package: 'randomForest'

The following object is masked from 'package:rattle':

importance

The following object is masked from 'package:dplyr':

combine

corrplot 0.84 loaded

Loading required package: lattice

Loading required package: ggplot2

Attaching package: 'ggplot2'

The following object is masked from 'package:randomForest':

margin

Loaded gbm 2.1.8

# Loading the Data

```{r}
TrainData <- read.csv(url("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"),header=TRUE)
print(dim(TrainData))
```

[1] 19622   160

```{r}
ValidData <- read.csv(url("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"),header=TRUE)
print(dim(ValidData))
```

[1]  20 160

```{r}
print(str(TrainData))
```

'data.frame':    19622 obs. of  160 variables:

$ X                       : int  1 2 3 4 5 6 7 8 9 10 ...

$ user_name               : chr  "carlitos" "carlitos" "carlitos" "carlitos" ...

$ raw_timestamp_part_1    : int  1323084231 1323084231 1323084231 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 1323084232 ...

$ raw_timestamp_part_2    : int  788290 808298 820366 120339 196328 304277 368296 440390 484323 484434 ...

$ cvtd_timestamp          : chr  "05/12/2011 11:23" "05/12/2011 11:23" "05/12/2011 11:23" "05/12/2011 11:23" ...

$ new_window              : chr  "no" "no" "no" "no" ...

$ num_window              : int  11 11 11 12 12 12 12 12 12 12 ...

$ roll_belt               : num  1.41 1.41 1.42 1.48 1.48 1.45 1.42 1.42 1.43 1.45 ...

$ pitch_belt              : num  8.07 8.07 8.07 8.05 8.07 8.06 8.09 8.13 8.16 8.17 ...

$ yaw_belt                : num  -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 -94.4 ...

$ total_accel_belt        : int  3 3 3 3 3 3 3 3 3 3 ...

$ kurtosis_roll_belt      : chr  "" "" "" "" ...

$ kurtosis_picth_belt     : chr  "" "" "" "" ...

$ kurtosis_yaw_belt       : chr  "" "" "" "" ...

$ skewness_roll_belt      : chr  "" "" "" "" ...

$ skewness_roll_belt.1    : chr  "" "" "" "" ...

$ skewness_yaw_belt       : chr  "" "" "" "" ...

$ max_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...

$ max_picth_belt          : int  NA NA NA NA NA NA NA NA NA NA ...

$ max_yaw_belt            : chr  "" "" "" "" ...

$ min_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...

$ min_pitch_belt          : int  NA NA NA NA NA NA NA NA NA NA ...

$ min_yaw_belt            : chr  "" "" "" "" ...

$ amplitude_roll_belt     : num  NA NA NA NA NA NA NA NA NA NA ...

$ amplitude_pitch_belt    : int  NA NA NA NA NA NA NA NA NA NA ...

$ amplitude_yaw_belt      : chr  "" "" "" "" ...

$ var_total_accel_belt    : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_roll_belt        : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_roll_belt           : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_pitch_belt       : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_pitch_belt          : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_yaw_belt         : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_yaw_belt            : num  NA NA NA NA NA NA NA NA NA NA ...

$ gyros_belt_x            : num  0 0.02 0 0.02 0.02 0.02 0.02 0.02 0.02 0.03 ...

$ gyros_belt_y            : num  0 0 0 0 0.02 0 0 0 0 0 ...

$ gyros_belt_z            : num  -0.02 -0.02 -0.02 -0.03 -0.02 -0.02 -0.02 -0.02 -0.02 0 ...

$ accel_belt_x            : int  -21 -22 -20 -22 -21 -21 -22 -22 -20 -21 ...

$ accel_belt_y            : int  4 4 5 3 2 4 3 4 2 4 ...

$ accel_belt_z            : int  22 22 23 21 24 21 21 21 24 22 ...

$ magnet_belt_x           : int  -3 -7 -2 -6 -6 0 -4 -2 1 -3 ...

$ magnet_belt_y           : int  599 608 600 604 600 603 599 603 602 609 ...

$ magnet_belt_z           : int  -313 -311 -305 -310 -302 -312 -311 -313 -312 -308 ...

$ roll_arm                : num  -128 -128 -128 -128 -128 -128 -128 -128 -128 -128 ...

$ pitch_arm               : num  22.5 22.5 22.5 22.1 22.1 22 21.9 21.8 21.7 21.6 ...

$ yaw_arm                 : num  -161 -161 -161 -161 -161 -161 -161 -161 -161 -161 ...

$ total_accel_arm         : int  34 34 34 34 34 34 34 34 34 34 ...

$ var_accel_arm           : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_roll_arm         : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_pitch_arm        : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...

$ avg_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...

$ stddev_yaw_arm          : num  NA NA NA NA NA NA NA NA NA NA ...

$ var_yaw_arm             : num  NA NA NA NA NA NA NA NA NA NA ...

$ gyros_arm_x             : num  0 0.02 0.02 0.02 0 0.02 0 0.02 0.02 0.02 ...

$ gyros_arm_y             : num  0 -0.02 -0.02 -0.03 -0.03 -0.03 -0.03 -0.02 -0.03 -0.03 ...

$ gyros_arm_z             : num  -0.02 -0.02 -0.02 0.02 0 0 0 0 -0.02 -0.02 ...

$ accel_arm_x             : int  -288 -290 -289 -289 -289 -289 -289 -289 -288 -288 ...

$ accel_arm_y             : int  109 110 110 111 111 111 111 111 109 110 ...

$ accel_arm_z             : int  -123 -125 -126 -123 -123 -122 -125 -124 -122 -124 ...

$ magnet_arm_x            : int  -368 -369 -368 -372 -374 -369 -373 -372 -369 -376 ...

$ magnet_arm_y            : int  337 337 344 344 337 342 336 338 341 334 ...

$ magnet_arm_z            : int  516 513 513 512 506 513 509 510 518 516 ...

$ kurtosis_roll_arm       : chr  "" "" "" "" ...

$ kurtosis_picth_arm      : chr  "" "" "" "" ...

$ kurtosis_yaw_arm        : chr  "" "" "" "" ...

$ skewness_roll_arm       : chr  "" "" "" "" ...

$ skewness_pitch_arm      : chr  "" "" "" "" ...

$ skewness_yaw_arm        : chr  "" "" "" "" ...

$ max_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...

$ max_picth_arm           : num  NA NA NA NA NA NA NA NA NA NA ...

$ max_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...

$ min_roll_arm            : num  NA NA NA NA NA NA NA NA NA NA ...

$ min_pitch_arm           : num  NA NA NA NA NA NA NA NA NA NA ...

$ min_yaw_arm             : int  NA NA NA NA NA NA NA NA NA NA ...

$ amplitude_roll_arm      : num  NA NA NA NA NA NA NA NA NA NA ...

$ amplitude_pitch_arm     : num  NA NA NA NA NA NA NA NA NA NA ...

$ amplitude_yaw_arm       : int  NA NA NA NA NA NA NA NA NA NA ...

$ roll_dumbbell           : num  13.1 13.1 12.9 13.4 13.4 ...

$ pitch_dumbbell          : num  -70.5 -70.6 -70.3 -70.4 -70.4 ...

$ yaw_dumbbell            : num  -84.9 -84.7 -85.1 -84.9 -84.9 ...

$ kurtosis_roll_dumbbell  : chr  "" "" "" "" ...

$ kurtosis_picth_dumbbell : chr  "" "" "" "" ...

$ kurtosis_yaw_dumbbell   : chr  "" "" "" "" ...

$ skewness_roll_dumbbell  : chr  "" "" "" "" ...

$ skewness_pitch_dumbbell : chr  "" "" "" "" ...

$ skewness_yaw_dumbbell   : chr  "" "" "" "" ...

$ max_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...

$ max_picth_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...

$ max_yaw_dumbbell        : chr  "" "" "" "" ...

$ min_roll_dumbbell       : num  NA NA NA NA NA NA NA NA NA NA ...

$ min_pitch_dumbbell      : num  NA NA NA NA NA NA NA NA NA NA ...

$ min_yaw_dumbbell        : chr  "" "" "" "" ...

$ amplitude_roll_dumbbell : num  NA NA NA NA NA NA NA NA NA NA ...

[list output truncated]

NULL

# Data Cleaning

```{r}
# Removing NA's, empty values, and unnecesary variables in the Trainning dataset.
EmptyCols <- which(colSums(is.na(TrainData) |TrainData=="")>0.9*dim(TrainData)[1]) 
TrainDataClean <- TrainData[,-EmptyCols]
TrainDataClean <- TrainDataClean[,-c(1:7)]
print(dim(TrainDataClean))
```

[1] 19622    53

```{r}
# Removing NA's, empty values in the Test dataset.
EmptyCols <- which(colSums(is.na(ValidData) |ValidData=="")>0.9*dim(ValidData)[1]) 
ValidDataClean <- ValidData[,-EmptyCols]
ValidDataClean <- ValidDataClean[,-1]
print(dim(ValidDataClean))
```

[1] 20 59

# Low variation data exclusion

```{r}
# Low variation data exclusion from the TrainData Dataset
NZV <- nearZeroVar(TrainDataClean)
print(NZV)
```

integer(0)

Acording to this, all the current variables report some variation. No more variables need to be removed from the training dataset.

# Data Partitioning for prediction

```{r}
set.seed(12345) 
inTrain <- createDataPartition(TrainDataClean$classe, p = 0.7, list = FALSE)
TrainData <- TrainDataClean[inTrain, ]
TestData <- TrainDataClean[-inTrain, ]
print(dim(TrainData))
```

[1] 13737    53

After cleaning, the new training data set has only 53 columns.

## Exploratoy Data Analysis

```{r}
corMatrix <- cor(TrainData[, -53])
corrplot(corMatrix, order = "FPC", method = "color", type = "lower", 
         tl.cex = 0.8, tl.col = rgb(0, 0, 0),mar = c(1, 1, 1, 1), title = "Training Dataset Correlogram")
```

![al image](http://example.com/logo.png)

All the correlations have a darker tone of blue if it’s closer to 1, and a darker tone of red when it’s closer to -1, which means a stronger relationship in both cases.

```{r}
# Count the number of variables that are highly correlated with another one
M <- abs(cor(TrainData[,-53])); diag(M) <- 0
M <- which(M > 0.8, arr.ind = T)
M <- dim(M)[1]
print(M)
```

[1] 38

There are 38 pairs of highly correlated variables. 

## Selection of the Predictive Model


To predict the outcome, we will use two different methods to model the regression using the TrainData dataset:
- Decision Trees
- Random Forest
Then, they will be applied to the TestData dataset to compare accuracies. The best model will be our Final model.


# Cross Validation

Cross-validation is done for each model with K = 3. 

```{r}
fitControl <- trainControl(method='cv', number = 3)
```

# Decision Trees Model

```{r}
# Decision Trees Model
DT_Model <- train(classe~., data=TrainData, method="rpart", trControl=fitControl)
#  Plot 
fancyRpartPlot(DT_Model$finalModel)
```

Now we validate the model to see it's performance by looking at the accuracy variable.

```{r}
# Testing the model
DT_Predict <- predict(DT_Model, newdata=TestData)
DT_cm <- confusionMatrix(DT_Predict, as.factor(TestData$classe))
# Display confusion matrix and model accuracy
print(DT_cm)
```

Confusion Matrix and Statistics

Reference

Prediction    A    B    C    D    E

A 1525  484  499  423  153

B   29  385   37  187  159

C  116  270  490  354  289

D    0    0    0    0    0

E    4    0    0    0  481

Overall Statistics                                           

Accuracy : 0.4895          

95% CI : (0.4767, 0.5024)

No Information Rate : 0.2845          

P-Value [Acc > NIR] : < 2.2e-16                                                  

Kappa : 0.3324                                                     

Mcnemar's Test P-Value : NA              

Statistics by Class:

Class: A Class: B Class: C Class: D Class: E

Sensitivity            0.9110  0.33802  0.47758   0.0000  0.44455

Specificity            0.6298  0.91319  0.78823   1.0000  0.99917

Pos Pred Value         0.4945  0.48306  0.32258      NaN  0.99175

Neg Pred Value         0.9468  0.85181  0.87723   0.8362  0.88870

Prevalence             0.2845  0.19354  0.17434   0.1638  0.18386

Detection Rate         0.2591  0.06542  0.08326   0.0000  0.08173

Detection Prevalence   0.5240  0.13543  0.25811   0.0000  0.08241

Balanced Accuracy      0.7704  0.62560  0.63291   0.5000  0.72186

```{r}
# Model Accuracy
print(DT_cm$overall[1])
```

 Accuracy 

0.4895497

Using cross-validation, the accuracy of this first model is about 0.489.Therefore the out-of-sample-error is 0.5, which is high.

## Random Forest Model

```{r}
# Random Forest Model
RF_Model <- train(classe~., data=TrainData, method="rf", trControl=fitControl, verbose=FALSE)
# Plot
plot(RF_Model,main="RF Model Accuracy by number of predictors")
```

Now we validate the model to see it's performance by looking at the accuracy variable.

```{r}
# Testing the model
RF_Predict <- predict(RF_Model, newdata=TestData)
RF_cm <- confusionMatrix(RF_Predict, as.factor(TestData$classe))
# Display confusion matrix and model accuracy
print(RF_cm)
```

Confusion Matrix and Statistics

Reference

Prediction    A    B    C    D    E

A 1673    6    0    0    0

B    1 1130    5    0    0

C    0    3 1018    8    2

D    0    0    3  955    1

E    0    0    0    1 1079

Overall Statistics

Accuracy : 0.9949          

95% CI : (0.9927, 0.9966)

No Information Rate : 0.2845          

P-Value [Acc > NIR] : < 2.2e-16       

Kappa : 0.9936          

Mcnemar's Test P-Value : NA              

Statistics by Class:

Class: A Class: B Class: C Class: D Class: E

Sensitivity            0.9994   0.9921   0.9922   0.9907   0.9972

Specificity            0.9986   0.9987   0.9973   0.9992   0.9998

Pos Pred Value         0.9964   0.9947   0.9874   0.9958   0.9991

Neg Pred Value         0.9998   0.9981   0.9984   0.9982   0.9994

Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839

Detection Rate         0.2843   0.1920   0.1730   0.1623   0.1833

Detection Prevalence   0.2853   0.1930   0.1752   0.1630   0.1835

Balanced Accuracy      0.9990   0.9954   0.9948   0.9949   0.9985

```{r}
# Model Accuracy
print(RF_cm$overall[1])
```

Accuracy 

0.9949023

Using cross-validation, the model accuracy is 0.994. Therefore the out-of-sample-error is 0.006.

```{r}
plot(RF_Model$finalModel,main="Model error of Random forest model by number of trees")
```

## Prediction of the values of classe for the validation data

By looking all  the accuracy rate values of the models, we select ‘Random Forest’ model. 

```{r}
# Model Validation 
Prediction_Test <- predict(RF_Model,newdata=ValidDataClean)
print(Prediction_Test)
```

[1] B A B A A E D B A A B C B A E E A B B B

Levels: A B C D E

## Conclusion


The confusion matrices shows that the Random Forest algorithm performs better than decision trees. The accuracy for the Random Forest model is 0.994 compared to 0.489 for Decision Tree model. Hence our selected predictive model is Random Forest model.
