# Machine Learning - Prediction Assignment
Johnson Joseph  
February 27, 2016  
  Six young health participants were asked to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions: exactly according to the specification (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).
  The goal of the project is to predict the manner in which the participants did the exercise. This is the "classe" variable in the data set. Also, lets split the training data into two sets, one for training and other for testing. The small data set provided will be used for the verification. 
  
### Data Cleaning
     Let's perform some initial data cleaning after looking at the data. There are lot of columns that contain only NA values and these columns are not needed in the prediction model. So, lets remove those columns. Also, columns 1 to 7 are details about the user and experiment time. These columns also should not be included in the model.Let's exclude those columns and build a clean training dataset.


```r
set.seed(3842)
library(caret)
library(dplyr)
library(randomForest)

download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", destfile = "ML-TrainingData.csv", method = "curl")
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", destfile = "ML-TestingData.csv", method = "curl")

trainingData <- read.csv("ML-TrainingData.csv",header = TRUE, na.strings = c("NA","#DIV/0!"))
trainingData <- trainingData[, apply(trainingData, 2, function(x) !any(is.na(x)))]
trainingData <- trainingData[, -(1:7)] 

verifyData <- read.csv("ML-TestingData.csv",header = TRUE, na.strings = c("NA","#DIV/0!"))
verifyData <- verifyData[, apply(verifyData, 2, function(x) !any(is.na(x)))]
verifyData <- verifyData[, -c(1:7,60)] 

trainIdx<-createDataPartition(y=trainingData$classe, p=0.75,list=F)
training<-trainingData[trainIdx,] 
testing<-trainingData[-trainIdx,] 
```

Lets choose a Random Forest with cross validation as the modeling algoritham.Then we will train the model using the training dataset. We will apply the model to the test dataset and measure how good the model fits with the test data.


```r
library(caret)
trainCtrl <- trainControl(method = "cv", number = 5, allowParallel = TRUE, verboseIter = TRUE)
exerciseModel <- train(classe ~ ., data = training, method = "rf", trControl=trainCtrl)
```

```
## + Fold1: mtry= 2 
## - Fold1: mtry= 2 
## + Fold1: mtry=27 
## - Fold1: mtry=27 
## + Fold1: mtry=52 
## - Fold1: mtry=52 
## + Fold2: mtry= 2 
## - Fold2: mtry= 2 
## + Fold2: mtry=27 
## - Fold2: mtry=27 
## + Fold2: mtry=52 
## - Fold2: mtry=52 
## + Fold3: mtry= 2 
## - Fold3: mtry= 2 
## + Fold3: mtry=27 
## - Fold3: mtry=27 
## + Fold3: mtry=52 
## - Fold3: mtry=52 
## + Fold4: mtry= 2 
## - Fold4: mtry= 2 
## + Fold4: mtry=27 
## - Fold4: mtry=27 
## + Fold4: mtry=52 
## - Fold4: mtry=52 
## + Fold5: mtry= 2 
## - Fold5: mtry= 2 
## + Fold5: mtry=27 
## - Fold5: mtry=27 
## + Fold5: mtry=52 
## - Fold5: mtry=52 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 2 on full training set
```

We will predit the result of the test data using the random forest model. The predicted values are compared with the actual values to see the accuracy of the model fit.


```r
result <- predict(exerciseModel, newdata = testing)
confusionMatrix(result,testing$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1394    7    0    0    0
##          B    1  941    7    0    0
##          C    0    1  847   13    0
##          D    0    0    1  791    1
##          E    0    0    0    0  900
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9937         
##                  95% CI : (0.991, 0.9957)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : < 2.2e-16      
##                                          
##                   Kappa : 0.992          
##  Mcnemar's Test P-Value : NA             
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9993   0.9916   0.9906   0.9838   0.9989
## Specificity            0.9980   0.9980   0.9965   0.9995   1.0000
## Pos Pred Value         0.9950   0.9916   0.9837   0.9975   1.0000
## Neg Pred Value         0.9997   0.9980   0.9980   0.9968   0.9998
## Prevalence             0.2845   0.1935   0.1743   0.1639   0.1837
## Detection Rate         0.2843   0.1919   0.1727   0.1613   0.1835
## Detection Prevalence   0.2857   0.1935   0.1756   0.1617   0.1835
## Balanced Accuracy      0.9986   0.9948   0.9936   0.9917   0.9994
```

Now, lets predit the results of the validation dataset. The results show that the prediction has an accuracy of 99.3%. 


```r
predictValidation <- predict(exerciseModel,newdata = verifyData)
predictValidation
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

