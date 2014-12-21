### Introduction

This file describes the work that has been performed to clean up the data.


#### Load test and training sets and the activities

The data set has been stored in the `UCI HAR Dataset/` directory.

In case the previously mentioned directory do no exists, the original file will be downloaded and extracted
The `unzip` function is used to extract the zip file in this directory.
```
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(fileUrl, destfile = "Dataset.zip", method = "curl")
unzip("Dataset.zip")
```

`read.table` is used to load the data to R environment for the data, the activities and the subject of both test and training datasets.

```
testData <- read.table("./UCI HAR Dataset/test/X_test.txt",header=FALSE)
testData_act <- read.table("./UCI HAR Dataset/test/y_test.txt",header=FALSE)
testData_sub <- read.table("./UCI HAR Dataset/test/subject_test.txt",header=FALSE)
trainData <- read.table("./UCI HAR Dataset/train/X_train.txt",header=FALSE)
trainData_act <- read.table("./UCI HAR Dataset/train/y_train.txt",header=FALSE)
trainData_sub <- read.table("./UCI HAR Dataset/train/subject_train.txt",header=FALSE)
```

#### Descriptive activity names to name the activities in the data set

The class labels linked with their activity names are loaded from the `activity_labels.txt` file. The numbers of the `testData_act` and `trainData_act` data frames are replaced by those names:

```
activities <- read.table("./UCI HAR Dataset/activity_labels.txt",header=FALSE,colClasses="character")
testData_act$V1 <- factor(testData_act$V1,levels=activities$V1,labels=activities$V2)
trainData_act$V1 <- factor(trainData_act$V1,levels=activities$V1,labels=activities$V2)
```

#### Appropriately labels the data set with descriptive activity names

Each data frame of the data set is labeled - using the `features.txt` - with the information about the variables used on the feature vector. The `Activity` and `Subject` columns are also named properly before merging them to the test and train dataset.

```
features <- read.table("./UCI HAR Dataset/features.txt",header=FALSE,colClasses="character")
colnames(testData)<-features$V2
colnames(trainData)<-features$V2
colnames(testData_act)<-c("Activity")
colnames(trainData_act)<-c("Activity")
colnames(testData_sub)<-c("Subject")
colnames(trainData_sub)<-c("Subject")
```

#### Merge test and training sets into one data set, including the activities

The `Activity` and `Subject` columns are appended to the test and train data frames, and then are both merged in the `bigData` data frame.

```
testData<-cbind(testData,testData_act)
testData<-cbind(testData,testData_sub)
trainData<-cbind(trainData,trainData_act)
trainData<-cbind(trainData,trainData_sub)
bigData<-rbind(testData,trainData)
```

#### Extract only the measurements on the mean and standard deviation for each measurement

`mean()` and `sd()` are used against `bigData` via `sapply()` to extract the requested measurements.

```
bigData_mean<-sapply(bigData,mean,na.rm=TRUE)
bigData_sd<-sapply(bigData,sd,na.rm=TRUE)
```

A warning is returned for the `Activity` column because it's not numeric. This does not impact the calcucation of the rest and NA is stored in the new data frames instead, since mean and sd are not applicable in this case. The same applies for `Subject` where we're not interested about the mean and sd, but since it's numeric already there is no warning.

#### Creates a second, independent tidy data set with the average of each variable for each activity and each subject.

Finaly the desired result, a `tidy` data table is created with the average of each measurement per activity/subject combination. The new dataset is saved in `tidy.txt` file.

```
DT <- data.table(bigData)
tidy<-DT[,lapply(.SD,mean),by="Activity,Subject"]
write.table(tidy,file="tidy.txt",sep=",",col.names = NA)
```