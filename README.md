---
title: "Practical Machine Learning Project"
author: "Rajesh Menon"
date: "Monday, April 20, 2015"
output: html_document
---

###Background
  
  Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset). 



###Data 


The training data for this project are available here: 
  
  https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here: 
  
  https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

The data for this project come from this source: http://groupware.les.inf.puc-rio.br/har. If you use the document you create for this class for any purpose please cite them as they have been very generous in allowing their data to be used for this kind of assignment. 

###Objective

The goal of your project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. You may use any of the other variables to predict with. You should create a report describing how you built your model, how you used cross validation, what you think the expected out of sample error is, and why you made the choices you did. You will also use your prediction model to predict 20 different test cases. 


### Loading and Cleaning the data
Loading the training data set 

```{r }
urlTrain <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
trainingset <- read.csv(urlTrain, na.strings=c("NA","#DIV/0!", ""))
```

Loading the testing data set 

```{r }
urlTest <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
testingset <- read.csv(urlTest, na.strings=c("NA","#DIV/0!", ""))
```

Delete columns with all missing values

```{r }
trainingset<-trainingset[,colSums(is.na(trainingset)) == 0]
testingset <-testingset[,colSums(is.na(testingset)) == 0]
```

variable like: id, timestamps, individuals’ names, etc. are not suitable to be used in prediction and are removed

```{r }
trainingset   <-trainingset[,-c(1:7)]
testingset <-testingset[,-c(1:7)]
library(randomForest)
library(caret)
library(e1071)
```

### Cross Validation

```{r }
set.seed(14424)
```

In order to avoid overfitting and to reduce out of sample errors, TrainControl is used to perform 5-fold cross validation.
Also, PCA will be used in the pre-processing to eliminate highly correlated variables. 

```{r }
tc <- trainControl(method = "cv", number = 5, verboseIter=FALSE , preProcOptions="pca", allowParallel=TRUE)
```

Three models are estimated: Random forest, Support Vector Machine (radial) and a Logit Boosted mode

```{r }
rf <- train(classe ~ ., data = trainingset, method = "rf", trControl= tc)
svmr <- train(classe ~ ., data = trainingset, method = "svmRadial", trControl= tc)
logitboost <- train(classe ~ ., data = trainingset, method = "LogitBoost", trControl= tc)
model <- c("Random Forest", "SVM (radial)","LogitBoost")
Accuracy <- c(max(rf$results$Accuracy),
              max(svmr$results$Accuracy),
              max(logitboost$results$Accuracy)
              )
performance <- cbind(model,Accuracy)
knitr::kable(performance)
```

From the above, it looks like Random forest and SVM(radial) provide the best results
with Random Forest giving an out-of-sample error of 0.005 (which is 1 minus 0.995) 

### Prediction
Let's predict with the best models noted earlier, Random Forest 

```{r }
rfPred <- predict(rf, testingset)
pml_write_files = function(x){
n = length(x)
for(i in 1:n){
filename = paste0("problem_id_",i,".txt")
write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
}
}
pml_write_files(rfPred)
```


---
title: "answer_files.R"
author: "rstudio"
date: "Sat Apr 25 00:24:23 2015"
