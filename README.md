# Pratical-Machine-learning-course-project
---
title: "Pratical Machine Learning-Course Projet"
author: "Isaac N Beas"
date: "10/8/2016"
output: html_document
---

## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 
In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants.

## Library used for data processing 

```{r}
library(rpart)
library(rpart.plot)
library(corrplot)
library(caret)
library(randomForest)
library(e1071)
```

## Load data 


```{r}
trainUrl <-"https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
testUrl <- "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
trainFile <- "./data/pml-training.csv"
testFile  <- "./data/pml-testing.csv"
if (!file.exists("./data")) {
   dir.create("./data")
}
if (!file.exists(trainFile)) {
   download.file(trainUrl, destfile=trainFile, method="curl")
}
if (!file.exists(testFile)) {
   download.file(testUrl, destfile=testFile, method="curl")
}
```

## Read data 

we will read this 2 files into 2 data frames

```{r}
trainRaw <- read.csv("./data/pml-training.csv")
testRaw <- read.csv("./data/pml-testing.csv")
dim(trainRaw)
```

```{r}
dim(testRaw)
```

- training data set have 19622 observation and 160 variables
- testing data set contains 20 observations and 160 variables.
- the outcome to predict is the classe in the data training set

## Cleaning the data 
we will remove NA values in the dataset also variable with zero variance and ID
we remove column who not contribute to the measurement of the accelerometer
```{r}
sum(complete.cases(trainRaw))
trainRaw <- trainRaw[, colSums(is.na(trainRaw)) == 0]
testRaw <- testRaw[, colSums(is.na(testRaw)) == 0]
classe <- trainRaw$classe
trainRemove <- grepl("^X|timestamp|window", names(trainRaw))
trainRaw <- trainRaw[, !trainRemove]
trainCleaned <- trainRaw[, sapply(trainRaw, is.numeric)]
trainCleaned$classe <- classe
testRemove <- grepl("^X|timestamp|window", names(testRaw))
testRaw <- testRaw[, !testRemove]
testCleaned <- testRaw[, sapply(testRaw, is.numeric)]
```

- Now the training data have 19622 observations an 20 variables
- the testing data contain 20 observations and 53 variables

## split cleaning data 
we are puting trainig data set 70%
validation data set 30%

```{r}
set.seed(22519)
inTrain <- createDataPartition(trainCleaned$classe, p=0.70, list=F)
trainData <- trainCleaned[inTrain, ]
testData <- trainCleaned[-inTrain, ]
```

## Data model selection 
we will fit a predictive model with "Random Forest" who will select important variables automatically 
```{r}
controlRf <- trainControl(method="cv", 5)
modelRf <- train(classe ~ .,data=trainData, method="rf",trControl=controlRf, ntree=250)
modelRf
```

We can now estimate the performance of the model on the validation data set

```{r}
predictRf <- predict(modelRf, testData)
confusionMatrix(testData$classe, predictRf)
```

```{r}
accuracy <- postResample(predictRf, testData$classe)
accuracy
```

```{r}
viiir <- 1 - as.numeric(confusionMatrix(testData$classe, predictRf)$overall[1])
viiir
```
The estimated accuracy of the model = 99.35%
estimated out of sample error = 0.70 %

## Prediction Model
```{r}
result <- predict(modelRf, testCleaned[, -length(names(testCleaned))])
result
```

## Appendix Plotting matrix results 

1- Correlation 

```{r}
corrPlot <- cor(trainData[, -length(names(trainData))])
corrplot(corrPlot, method="color")
```

2- Decision Tree

```{r}
treeModel <- rpart(classe ~ ., data=trainData, method="class")
prp(treeModel)
```
