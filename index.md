Analysis for the PML assignment.
========================================================

The goal is to predict 'classe' variable, based on any set of predictors and to apply the predictive model to the set of test cases. A quick look at the test cases shows, that the predictors used in the original paper (averages etc.) cannot be used in this case, becasue the values of all of them are not defined for test cases.

Having no domain knowledge about the predictors I decided to use all available 'raw' variables and random forest as my classifier. The choise of classifier is dictated by the ease of use for mutli-class classification and its robustness for predictors of unknow distributions.


```r
# Construct names of all variables of interest:
ryp_components <- expand.grid(c("roll", "yaw", "pitch"), c("dumbbell", "arm", 
    "forearm", "belt"))
gam_components <- expand.grid(c("gyros", "accel", "magnet"), c("dumbbell", "arm", 
    "forearm", "belt"), c("x", "y", "z"))
ryp_predictors <- paste(ryp_components[[1]], ryp_components[[2]], sep = "_")
gam_predictors <- paste(gam_components[[1]], gam_components[[2]], gam_components[[3]], 
    sep = "_")
all_predictors <- c(ryp_predictors, gam_predictors)
```



```r
# Create formula to predict 'classe' based on all predictors.
fmla <- as.formula(paste("classe ~ ", paste(all_predictors, collapse = "+")))
```


My model is based on the following formula: 


```r
print(fmla)
```

```
## classe ~ roll_dumbbell + yaw_dumbbell + pitch_dumbbell + roll_arm + 
##     yaw_arm + pitch_arm + roll_forearm + yaw_forearm + pitch_forearm + 
##     roll_belt + yaw_belt + pitch_belt + gyros_dumbbell_x + accel_dumbbell_x + 
##     magnet_dumbbell_x + gyros_arm_x + accel_arm_x + magnet_arm_x + 
##     gyros_forearm_x + accel_forearm_x + magnet_forearm_x + gyros_belt_x + 
##     accel_belt_x + magnet_belt_x + gyros_dumbbell_y + accel_dumbbell_y + 
##     magnet_dumbbell_y + gyros_arm_y + accel_arm_y + magnet_arm_y + 
##     gyros_forearm_y + accel_forearm_y + magnet_forearm_y + gyros_belt_y + 
##     accel_belt_y + magnet_belt_y + gyros_dumbbell_z + accel_dumbbell_z + 
##     magnet_dumbbell_z + gyros_arm_z + accel_arm_z + magnet_arm_z + 
##     gyros_forearm_z + accel_forearm_z + magnet_forearm_z + gyros_belt_z + 
##     accel_belt_z + magnet_belt_z
```



```
## Loading required package: lattice
## Loading required package: ggplot2
```

I split the available training data into two random partitions: 75% for the traning set and 25% for the validation set used for the final estimation of out of sample accuracy of the model.

I use caret package to train randomForest on the traning set, and optimize the 'mtry' parameter (the number of variables to sample at each split) with cross-validation to maximize accuracy. I use the "oob" (out of bag) method which is avaible for random forest and provide fast and reliable estimate of the out of sample error for each value of the paramerter.


```r
trControl <- trainControl(verboseIter = TRUE, method = "oob")
rfModel <- train(fmla, data = trainSet, method = "rf", importance = FALSE, ntrees = 50, 
    trControl = trControl, tuneGrid = data.frame(mtry = c(4, 5, 6)))
```

```
## Loading required package: randomForest
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
```

```
## + : mtry=4
```

```
## Loading required namespace: e1071
```

```
## - : mtry=4 
## + : mtry=5 
## - : mtry=5 
## + : mtry=6 
## - : mtry=6 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 5 on full training set
```


Here are the cross validation out-of-sample accuracies for tested 'mtry' values:

```r
print(rfModel$results)
```

```
##   Accuracy  Kappa mtry
## 1   0.9953 0.9941    4
## 2   0.9954 0.9942    5
## 3   0.9951 0.9938    6
```

For the optimal value of mtry=5 the cross validation out of sample accuracy is: 0.9954

I use the validation set (holdout sample) to estimate the out of sample accuracy of the final model. 


```r
predicted <- predict(rfModel, newdata = testSet)
cm <- confusionMatrix(predicted, testSet$classe)
print(cm)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1395    6    0    0    0
##          B    0  942    4    0    0
##          C    0    1  848    7    0
##          D    0    0    3  797    4
##          E    0    0    0    0  897
## 
## Overall Statistics
##                                         
##                Accuracy : 0.995         
##                  95% CI : (0.992, 0.997)
##     No Information Rate : 0.284         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.994         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             1.000    0.993    0.992    0.991    0.996
## Specificity             0.998    0.999    0.998    0.998    1.000
## Pos Pred Value          0.996    0.996    0.991    0.991    1.000
## Neg Pred Value          1.000    0.998    0.998    0.998    0.999
## Prevalence              0.284    0.194    0.174    0.164    0.184
## Detection Rate          0.284    0.192    0.173    0.163    0.183
## Detection Prevalence    0.286    0.193    0.175    0.164    0.183
## Balanced Accuracy       0.999    0.996    0.995    0.995    0.998
```


Out of sample accuracy calculated on the holdout sample is: 0.9949
