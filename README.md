# Getting-and-Cleaning-Data-Course-Project

The original data set used for this project is:
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

More detailed description of the data is here: http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

The expected result is to create a tidy independent data set from the data above with the average of each variable for each activity and each subject.

The CodeBook.md contains the description of variables, data and the transformations made to the original dataset - 
https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip

The file run_analysis.R contains the script used to create the tidy dataset.

Here is how the script works:
## Download the Zip file
URL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(URL, destfile = "./DATA.zip", method = "curl")

## Unzip the file
unzip(zipfile= "./DATA.zip",exdir="./DATA")

## Step 1: Merges the training and test data sets

## a) Read the files
Path <- file.path("./DATA" , "UCI HAR Dataset")
files <- list.files(Path, recursive=TRUE)

## Activity files
ATest  <- read.table(file.path(Path, "test" , "Y_test.txt" ),header = FALSE)
ATrain <- read.table(file.path(Path, "train", "Y_train.txt"),header = FALSE)

## Subject Files
STrain <- read.table(file.path(Path, "train", "subject_train.txt"),header = FALSE)
STest  <- read.table(file.path(Path, "test" , "subject_test.txt"),header = FALSE)

## Feature files
FTest  <- read.table(file.path(Path, "test" , "X_test.txt" ),header = FALSE)
FTrain <- read.table(file.path(Path, "train", "X_train.txt"),header = FALSE)

## b) The actual merge

## merge by rows
Activity<- rbind(ATrain, ATest)
Subject <- rbind(STrain, STest)
Feature <- rbind(FTrain, FTest)

## set names to the partially merged values
names(Subject) <- c("subject")
names(Activity) <- c("activity")
FNames <- read.table(file.path(Path, "features.txt"),head=FALSE)
names(Feature)<- FNames$V2

## merge by columns
Merge <- cbind(Subject, Activity)
MergedData <- cbind(Feature, Merge)

## Step - 2 Extract only the measurements on the mean and standard deviation for each measurement.

## determine which columns contain "mean()" or "std()"
MSColsT <- FNames$V2[grep("mean\\(\\)|std\\(\\)", FNames$V2)]

MSCols <- c(as.character(MSColsT), "subject", "activity" )

## Extract the necessary columns
MergedData <- MergedData[, MSCols]

## Step 3 Uses descriptive activity names to name the activities in the data set
Label <- read.table(file.path(Path, "activity_labels.txt"),header = FALSE)

## Step-4 Appropriately label the data set with descriptive variable names


names(MergedData)<-gsub("^t", "time", names(MergedData))
names(MergedData) <-gsub("^f", "frequency", names(MergedData))
names(MergedData)<-gsub("Acc", "Accelerometer", names(MergedData))
names(MergedData)<-gsub("Gyro", "Gyroscope", names(MergedData))
names(MergedData)<-gsub("Mag", "Magnitude", names(MergedData))
names(MergedData)<-gsub("BodyBody", "Body", names(MergedData))

## Step 5 From the data set in step 4, tidy data set with the average of each variable for each activity and each subject.
library(plyr);
Data2 <- aggregate(. ~subject + activity, MergedData, mean )
TidyData <- Data2 [order(Data2$subject,Data2$activity),]
write.table(TidyData, file = "tidydata.txt",row.name=FALSE) 
