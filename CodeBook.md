library(data.table)
library(dplyr)

# 0. load test and training sets and the activities
if (!file.exists("UCI HAR Dataset")){
    fileUrl <- "http://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
    download.file(fileUrl, destfile = "Dataset.zip")
    unzip("Dataset.zip")
}

### PART 1. Merges the training and the test sets to create one data set.

# reading all the tables
tmp_testData <- read.table("./UCI HAR Dataset/test/X_test.txt",header=FALSE)
tmp_testData_act <- read.table("./UCI HAR Dataset/test/y_test.txt",header=FALSE)
tmp_testData_sub <- read.table("./UCI HAR Dataset/test/subject_test.txt",header=FALSE)
tmp_trainData <- read.table("./UCI HAR Dataset/train/X_train.txt",header=FALSE)
tmp_trainData_act <- read.table("./UCI HAR Dataset/train/y_train.txt",header=FALSE)
tmp_trainData_sub <- read.table("./UCI HAR Dataset/train/subject_train.txt",header=FALSE)
tmp_features <- read.table ("./UCI HAR Dataset/features.txt",header=FALSE,as.is=TRUE)


# merging corresponding tables
tmp_mergedData <- rbind(tmp_trainData,tmp_testData)
tmp_mergedData_act <- rbind(tmp_trainData_act,tmp_testData_act)
tmp_mergedData_sub <- rbind(tmp_trainData_sub,tmp_testData_sub)

# assigning the column headers
colnames(tmp_mergedData) <- tmp_features[,2]
colnames(tmp_mergedData_sub) <- c("Subject")
colnames(tmp_mergedData_act) <- c("Activity")

# combine all data into one file
allData <- cbind(tmp_mergedData,tmp_mergedData_sub,tmp_mergedData_act)

# cleaning the temporary files
rm(list=ls(pattern="tmp"))


## normally it should look like this
## Columns: variable names (from features.txt) (561), Subject (562), Activities (563)
## https://class.coursera.org/getdata-010/forum/thread?thread_id=49#post-1230

### END OF PART 1


### PART 2. Extracts only the measurements on the mean and standard deviation for each measurement.

# Duplicated columns exist in the complete data, let's eliminate them, assign to uniqueData
uniqueData<- allData[,!duplicated(colnames(allData))]

# use select() to get columns containing either "mean" or "std" assing to mean_stdData
mean_stdData <- select(uniqueData,contains("mean"),contains("std"))

### END OF PART 2.

### PART 3. Uses descriptive activity names to name the activities in the data set

# load activity labels
activity_labels <- read.table("./UCI HAR Dataset/activity_labels.txt")

# replace Activities coulumns with labels
uniqueData<-mutate(uniqueData,Activity= activity_labels[match(Activity,activity_labels[,1]),2])

### END OF PART 3.

### PART 4. Appropriately labels the data set with descriptive variable names. 
# Four major criteria based on lecture slide 16
# https://d396qusza40orc.cloudfront.net/getdata/lecture_slides/04_01_editingTextVariables.pdf
# - All lower case if possible
# - Descriptive
# - Not duplicated
# - No underscore, no white spaces, no dots

# At this stage it is required to my judgement to focus on the last point

# Elimination of underscores
names(uniqueData)<- gsub("_","",names(uniqueData))

# Elimination of white spaces
names(uniqueData)<- gsub(" ","",names(uniqueData))

# Elimination of ()
names(uniqueData)<- gsub("()","",names(uniqueData))

### END OF PART 4.

### PART 5. From the data set in step 4, creates a second, 
### independent tidy data set with the average of each variable for each activity and each subject. 
