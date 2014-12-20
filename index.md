Analysis for the PML assignment.
========================================================

Analysis for the PML project.


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
set.seed(199)
trainData <- read.csv("data/pml-training.csv")
testData <- read.csv("data/pml-testing.csv")
trainIdx <- createDataPartition(y = trainData$classe, p = 0.75, list = FALSE)
trainSet <- trainData[trainIdx, ]
testSet <- trainData[-trainIdx, ]
```



Construct names of all variables of interest:

```r
ryp_components <- expand.grid(c("roll", "yaw", "pitch"), c("dumbbell", "arm", 
    "forearm", "belt"))
ryp_predictors <- paste(ryp_components[[1]], ryp_components[[2]], sep = "_")
all_predictors <- ryp_predictors
```


Create formula to predict 'classe' based on all predictors:

```r
fmla <- as.formula(paste("classe ~ ", paste(all_predictors, collapse = "+")))
print(fmla)
```

```
## classe ~ roll_dumbbell + yaw_dumbbell + pitch_dumbbell + roll_arm + 
##     yaw_arm + pitch_arm + roll_forearm + yaw_forearm + pitch_forearm + 
##     roll_belt + yaw_belt + pitch_belt
```


Train a random forest model

```r
trControl <- trainControl(verboseIter = TRUE, method = "none")
rfModel <- train(fmla, data = trainSet, method = "rf", importance = TRUE, ntrees = 200, 
    trControl = trControl, tuneGrid = data.frame(mtry = 5), tuneLength = 1)
```

```
## Loading required package: randomForest
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
## Loading required namespace: e1071
```

```
## Fitting mtry = 5 on full training set
```


You can also embed plots, for example:


```r
varImpPlot(rfModel$finalModel)
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 


Calculate out of sample accuracy (on the test set):

```r
predicted <- predict(rfModel, newdata = testSet)
print(confusionMatrix(predicted, testSet$classe))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1390    8    0    0    0
##          B    2  927    3    2    0
##          C    0   12  844    7    3
##          D    3    2    7  795    4
##          E    0    0    1    0  894
## 
## Overall Statistics
##                                         
##                Accuracy : 0.989         
##                  95% CI : (0.986, 0.992)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.986         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.996    0.977    0.987    0.989    0.992
## Specificity             0.998    0.998    0.995    0.996    1.000
## Pos Pred Value          0.994    0.993    0.975    0.980    0.999
## Neg Pred Value          0.999    0.994    0.997    0.998    0.998
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.283    0.189    0.172    0.162    0.182
## Detection Prevalence    0.285    0.190    0.177    0.165    0.183
## Balanced Accuracy       0.997    0.988    0.991    0.992    0.996
```




