# Practical Machine Learning (Course Project)
Project for the class Practical Machine Learning in Coursera

##Introduction

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. 

##Loading and cleaning data

First of all, read .csv files into R using fread function from "data.table" library. It is necessary to take into account, that NA values represented in files such as "#DIV/0!", "NA" or spaces.

```{r read}
library(data.table)
pml.training <- fread("pml-training.csv", na.strings = c("#DIV/0!", "", "NA"))
pml.testing <- fread("pml-testing.csv", na.strings = c("#DIV/0!", "", "NA"))
```

Secondly, remove columns about user information and time information and column of indices, because these factors are not related to sensor information:

```{r removecols}
pml.training <- pml.training[,c("V1", "user_name", "raw_timestamp_part_1", "raw_timestamp_part_2",
                "cvtd_timestamp", "new_window", "num_window") := NULL]
```

Remove columns, which NA values more than not NA values:

```{r removeNAcols}
sumNAs <- apply(pml.training, 2, function(x) {sum(is.na(x))})
notremoveCol <- sumNAs < (nrow(pml.training) / 2)
pml.training <- pml.training[, notremoveCol, with = F]
```

Convert "classe" variable to factor type:

```{r factorclasse}
pml.training <- pml.training[, classe := as.factor(classe)]
```

##Preparing datasets for training

Create 70% for train dataset and 30% for test dataset:

```{r create datasets}
library(caret)
set.seed(88)
Index <- createDataPartition(y = pml.training$classe, p=0.7,list=FALSE)
train <- pml.training[Index]
test <- pml.training[-Index]
```

##Train the model

Train the model using random forest:

```{r trainmodel}
library(randomForest)
FitRF <- randomForest(classe ~ ., data=train, method="class")
```

Predict on test data and build confusion matrix:

```{r confmatr}
pred <- predict(FitRF, test)
confusionMatrix(pred, test$classe)
```

As seen by the result of the confusion matrix, the model is good and efficient because it has an accuracy of ~99% and very good sensitivity & specificity values on the testing dataset. 

##Submission

```{r subm}
ans <- predict(FitRF, pml.testing)
ans
```

This submission scores 100% (20/20).

