Practical Machine Project
=========================

Project Overview
----------------

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: <http://groupware.les.inf.puc-rio.br/har> (see the section on the Weight Lifting Exercise Dataset).

Loading the Data
----------------

We first load the training and testing csv files into R data frames.

``` r
trainingStage <- read.csv("pml-training.csv")
testingStage <- read.csv("pml-testing.csv")
dim(trainingStage)
```

    ## [1] 19622   160

``` r
dim(testingStage)
```

    ## [1]  20 160

A quick analysis of the training dataset shows that it has 160 variables, and 19622 rows.

The testing dataset has 160 variables, and 20 rows.

### Exploring and Cleaning the Data

In reviewing the data, and examining the 160 variables in the training set, it becomes apparent that there are many NAs and zeroes that can safely be removed.

Before continuing our analysis, we are going to remove these unnecessary variables that contain little value for the purpose of our analysis.

``` r
removeCol <- grepl("^X|^amplitude_|^avg_|^kurtosis_|^max_|^min_|^skewness_|^stddev_|^var_|timestamp|window", names(trainingStage))
trainingTrimmed <- trainingStage[,!removeCol]
```

We trim the unnecessary variables from the Testing set in a similar manner.

``` r
testingTrimmed <- testingStage[,!removeCol]
```

### Subdivide Training Set

We further subdivide the training data to allow for further testing of the model fit.

``` r
set.seed(61233)
inTrain <- createDataPartition(trainingTrimmed$classe, p=0.75, list=FALSE)
subTrainSet <- trainingTrimmed[inTrain,]
subTestSet <- trainingTrimmed[-inTrain,]
```

### Build Model

Due to it's general robustness, we build a machine Random Forest learning model to predict the manner in which the exercise was done.

``` r
ctrlRF <- trainControl(method="cv",3)
modFit <- train(classe ~ ., data=subTrainSet,method="rf", trControl=ctrlRF, ntree=100)
```

    ## Loading required package: randomForest

    ## Warning: package 'randomForest' was built under R version 3.2.5

    ## randomForest 4.6-12

    ## Type rfNews() to see new features/changes/bug fixes.

    ## 
    ## Attaching package: 'randomForest'

    ## The following object is masked from 'package:ggplot2':
    ## 
    ##     margin

``` r
modFit
```

    ## Random Forest 
    ## 
    ## 14718 samples
    ##    53 predictor
    ##     5 classes: 'A', 'B', 'C', 'D', 'E' 
    ## 
    ## No pre-processing
    ## Resampling: Cross-Validated (3 fold) 
    ## Summary of sample sizes: 9812, 9812, 9812 
    ## Resampling results across tuning parameters:
    ## 
    ##   mtry  Accuracy   Kappa    
    ##    2    0.9877021  0.9844414
    ##   29    0.9885175  0.9854725
    ##   57    0.9846447  0.9805725
    ## 
    ## Accuracy was used to select the optimal model using  the largest value.
    ## The final value used for the model was mtry = 29.

We then apply the training model to the validation set.

``` r
prediction <- predict(modFit,subTestSet)
confMatrix <- confusionMatrix(subTestSet$classe,prediction)
```

``` r
confMatrix$overall
```

    ##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
    ##      0.9938825      0.9922601      0.9912784      0.9958689      0.2867047 
    ## AccuracyPValue  McnemarPValue 
    ##      0.0000000            NaN

The accuracy of the model is given as 99.39%, and so the out of sample error is 0.61%.

Apply Prediction to Test Set
----------------------------

Finally, we apply the model fit to the test data set we've set aside set aside test set.

``` r
finalPredict <- predict(modFit, testingTrimmed)
finalPredict
```

    ##  [1] B A B A A E D B A A B C B A E E A B B B
    ## Levels: A B C D E
