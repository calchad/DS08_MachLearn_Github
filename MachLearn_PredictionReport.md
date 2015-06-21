# Model to predict quality of activity performance
Carolyn Chadwick  
Tuesday, 2 June 2015  

- Prediction Assignment for the Practical Machine Learning Module of the Data Science Specialization    
Writeup & submission instructions & evaluation criteria can be found in the accompanying MachLearn_Readme.md file
      



## Synopsis

Human activity recognition research to date has focussed on quantitative measurements (eg. type, timing & duration of activity). Qualitative assessment of "how well" an activity is performed has received relatively little attention. Feedback on quality of performance has potential uses in training contexts provided that   
- correct execution & deviations from that norm can be defined  
- execution mistakes can automatically & reliably recognized

This report investigates performance quality recognition via algorithm.
using a "Weight Lifting Exercise"" dataset collected from sensors on bodies of participants & their equipment.  Six men used a 1.25kg dumbbell to perform one set of 10 repetitions of the Unilateral Dumbbell Biceps Curl in five different fashions:  
- Class A: lift dumbbell correctly  
- Class B: throwing the elbows to the front  
- Class C: lifting the dumbbell only halfway  
- Class D: lowering the dumbbell only halfway   
- Class E: throwing the hips to the front   

## Data Processing




```r
## download training & testing data from the web to local files if necessary
## and read into data tables in memory

DestTrain <- paste(getwd(),"PMLTrain.csv",sep="/")
DestTest <- paste(getwd(),"PMLTest.csv",sep="/")
NAStr    <- c("", "NA", "N/A", "NULL","#DIV/0!")

setInternet2(TRUE)

if( !file(DestTrain)){
     URLTrain  <- 
     "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
     download.file(URLTrain, destfile=DestTrain)
     }
if( !file(DestTest)){
     URLTest <- 
     "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"
     download.file(URLTest, destfile=DestTest)
     }

DTTrain   <- read.table(DestTrain, sep=",", header=TRUE, na.strings=NAStr)
dim(DTTrain)
```

```
## [1] 19622   160
```

```r
DTTest   <- read.table(DestTest, sep=",", header=TRUE, na.strings=NAStr)
dim(DTTest)
```

```
## [1]  20 160
```





 



```r
sum(NACount>0)     ## no. of columns with missing values  
```

```
## [1] 100
```

```r
table(NACount)     ## table showing spread of no. of missing varialbes
```

```
## NACount
##     0 19216 19217 19218 19220 19221 19225 19226 19227 19248 19293 19294 
##    60    67     1     1     1     4     1     4     2     2     1     1 
## 19296 19299 19300 19301 19622 
##     2     1     4     2     6
```


```r
## remove NA & other unwanted cols
## renaming first column to classe in both data sets 

NAColNames <- NULL
for(i in 8:nCols-1){
 if(NACount[i]==0){
  NAColNames <- c(NAColNames, DTCols[i])
 }
}

DTTrain     <- DTTrain[c(colnames(DTTrain[nCols]),NAColNames)]
DTTest      <- DTTest[c(NAColNames)]
DTTest1     <- as.data.frame(rep(NA,length(DTTest[,2])))
colnames(DTTest1)<-colnames(DTTrain[1])
DTTest      <- cbind(DTTest1, DTTest)
dim(DTTrain)
```

```
## [1] 19622    54
```

```r
dim(DTTest)
```

```
## [1] 20 54
```



```r
## set random number generator seed to ensure reproducibility of results
set.seed(1)

## split training data into own 60% own training & 40% own testing sets
PartTrain <- createDataPartition(y=DTTrain$classe, p=0.6, list=FALSE)
OwnTrain  <- DTTrain[PartTrain,]
OwnTest   <- DTTrain[-PartTrain,]
dim(OwnTrain)
```

```
## [1] 11776    54
```

```r
dim(OwnTest)
```

```
## [1] 7846   54
```

## Build Random Forest Model 
to predict the activity class using own training set (60% of original training data)



```r
## create random forest model from variable columns left in data
## save it for later use & blank the following 2 code lines to save
## processing time
OwnTrainModel <- train(classe~., method="rf", data=OwnTrain)
saveRDS(OwnTrainModel, "OwnTrainModel.RDS")

## read in saved model data
OwnTrainModel <- readRDS("OwnTrainModel.RDS")
```

## Evaluate Model 
by seeing how well it predicts classes for activities in own test set (40% of original training data)


```r
## Accuracy as the percentage of correct predictions by the model 
##
mean(predict(OwnTrainModel, OwnTest) == OwnTest$classe) * 100
```

```
## [1] 99.77058
```



## Conclusion
We have built a model to predict exercise form based on movement data. We estimate the out of sample error to be .2% (1 - testing accuracy). This is a promising result regarding the use of machine learning to detect bad exercise form. It must be notes that what we are truly predicting here is the which of 5 predetermined supervised movements a subject is performing. So, although we estimate a very low out of sample error, we can expect the error of predicting bad form in real life situations to be higher.


```





## Citation: 
The weight lifting dataset was sourced from  
    http://groupware.les.inf.puc-rio.br/har  
courtesy of the authors of the following research paper:     
    "Velloso, E.; Bulling, A.; Gellersen, H.; Ugulino, W.; Fuks, H. 
    Qualitative   Activity Recognition of Weight Lifting Exercises. 
    Proceedings of 4th International Conference in Cooperation with SIGCHI  
    (Augmented Human '13) . Stuttgart, Germany: ACM SIGCHI, 2013."


