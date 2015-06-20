# Code Book

This document describes the code inside `run_analysis.R`.

The workflow is as follows:

* check the data if not found then download the data
* Read dataset files from UCI HAR directory to given same variable names. 
>suffixes are "train" and "test". 
>prefixes are "X", "y" and "subject".eg.
 + `UCI HAR Dataset/train/X_train.txt`
 + `UCI HAR Dataset/train/y_train.txt`
 + `UCI HAR Dataset/train/subject_train.txt`

* Merging the data tables with combined prefix
* Merging the activity labels and subjects to data tables ( variable names with activity and subject suffix)
* Manipulating data ( extrat mean() and sd() columns added -ms suffix in variable name)
* Create the tidy data table and writing it to txt file.

####Load the required libraries if not installed then install first
``` 
if(!require(dplyr)){
  install.packages("dplyr")
  library(dplyr)
}
if(!require(reshape2)){
  install.packages("reshape2")
  library(reshape2)
}
```
####check if the "UCI HAR Dataset" is available in the working directory if not then download and extract. 
```
subDir<-"UCI HAR Dataset"
if (!file.exists(subDir)){
#####create a tempfile to for the zip file
`temp <- tempfile()`
#####download the zip file
download.file("https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip",temp)
#####unzip the zip file
unzip(temp)
#####cleanup delete the zip tempfile
unlink(temp)
} # end if file.exits block
```
####Read the test and train dataset 2947 X 561 , 7352 X 561
```
X_test  <- tbl_df(read.table("./UCI HAR Dataset/test/X_test.txt"))
X_train <- tbl_df(read.table("./UCI HAR Dataset/train/X_train.txt"))
#####Merge the dataset into one dataset combined_data
combined_data    <- bind_rows(X_test,X_train)
```
```
####Read the activity number of test/train dataset
y_test  <- tbl_df(read.table("./UCI HAR Dataset/test/y_test.txt"))
y_train <- tbl_df(read.table("./UCI HAR Dataset/train/y_train.txt"))

#####Merge the activity data into one dataset combined_y
combined_y    <- bind_rows(y_test,y_train)
```

####Read the subjects doing the activity for test/train data 2947X1 , 7352 X1
```
subject_test     <- tbl_df(read.table("./UCI HAR Dataset/test/subject_test.txt"))
subject_train    <- tbl_df(read.table("./UCI HAR Dataset/train/subject_train.txt"))
#####Merge the suject data into one dataset combined_subject
combined_subject    <- bind_rows(subject_test,subject_train )
names(combined_subject) <- c("subject")
```

####Read Names of the activitity correponds to each activity number 6X1
```
activity_labels  <- read.table("./UCI HAR Dataset/activity_labels.txt")
```

####Uses descriptive activity names to lables the activities in the data set
```
combined_y_with_activity <- left_join(combined_y,activity_labels,by="V1")
names(combined_y_with_activity) <- c("activity_number","activity_labels")
```
#####Read the features details that contain the name of the column variable of dataset 561X1
```
features         <- read.table("./UCI HAR Dataset/features.txt")
```

###Extracts only the measurements on the mean and standard deviation for each measurement. mean() std()
```
colfeatures             <- lapply(features[2], as.character)
colnames(combined_data) <- unlist(colfeatures)
combined_data_mean      <-combined_data[,grep("-mean()", colnames(combined_data),fixed=TRUE)] 
combined_data_std       <-combined_data[,grep("-std()", colnames(combined_data),fixed=TRUE)] 

####Merging this subset data sets into one dataset combined_data_ms
combined_data_ms        <- bind_cols(combined_data_mean,combined_data_std)

####Combine the activity and subject tables with this new subset dataset
combined_data_ms_activities_subject <- bind_cols(combined_y_with_activity,combined_subject)
combined_data_ms_activities_subject <-bind_cols(combined_data_ms_activities_subject,combined_data_ms)
```
###Create independent tidy data set with the average of each variable for each activity and each subject.
```
melted <- melt(combined_data_ms_activities_subject, id.vars=c("activity_labels", "subject")) 

tidy_data=melted %>% group_by(activity_labels,subject) %>% summarise_each(funs(mean),value) %>% dcast(subject~activity_labels)

print(tidy_data)
```
###writing the tidy data into txt file
```                                                       
write.table(tidy_data ,file = "tidyData.txt",sep = "\t", row.name=FALSE)
```                      
 