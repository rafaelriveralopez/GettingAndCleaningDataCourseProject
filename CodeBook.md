#
#Load libraries
#
library("data.table")
library("dplyr")

#
#Preparing the target directory for storing data
#
if (!file.exists("GCD")) {
  dir.create("GCD")
}
if (!file.exists("GCD/Project")) {
  dir.create("GCD/Project")
}
#
#Download the zip file and extracting the files
#
path <- "./GCD/Project/"
dataset <- paste0(path,"dataset.zip")

if(!file.exists(dataset)){
  fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileUrl, destfile = dataset, method = "curl")
  unzip(dataset, exdir = path, unzip = "unzip")
}

pathds <- paste0(path,"UCI HAR Dataset/")

#STEP 1. Merge the training and the test sets to create one data set

#Using data frame for storing files
subjectTrain <- read.table(paste0(pathds,"train/subject_train.txt"))
subjectTest <- read.table(paste0(pathds,"test/subject_test.txt"))

xTrain <- read.table(paste0(pathds,"train/X_train.txt"))
yTrain <- read.table(paste0(pathds,"train/y_train.txt"))

xTest <- read.table(paste0(pathds,"test/X_test.txt"))
yTest <- read.table(paste0(pathds,"test/y_test.txt"))

# Merge training and test data
subject <- rbind(subjectTrain,subjectTest)
x <- rbind(xTrain,xTest)
y <- rbind(yTrain,yTest)

#Rename the column name of subject data 
subject <- rename(subject, subject = V1)

#Rename the column name of y data (activity label) 
y <- rename(y, activityNumber = V1)

#Merge columns of data
data <- cbind(x,subject,y)

#END OF STEP 1

#STEP 2. Extract only the measurements on the mean and standard deviation for each measurement

#Read feature description file
features <- read.table(paste(pathds,"features.txt",sep=""))

#Rename columns V1 and V2 in features data frame
features <- rename(features, featureNumber = V1, featureName=V2)

#Identify the feature names that contain "mean" or "std"
match<-grep("mean\\(\\)|std\\(\\)",features$"featureName")

#Select the rows that feature number is matching
features <- filter(features,features$"featureNumber" %in% match)

#Add column with column name in data 
features <- mutate(features,featureColumn = paste0("V",featureNumber))

#Select only columns of data that are in features with mean or std
select <- select(data,features$"featureNumber")

#Add subject and activity number for data set
#data <- mutate(select, subject = data$"subject", activityNumber= data$"activityNumber")
data <- mutate(subject = data$"subject", activityNumber= data$"activityNumber", select)

#END OF STEP 2

#STEP 3. Uses descriptive activity names to name the activities in the data set.

#Read activity labels file
activities <- read.table(paste0(pathds,"activity_labels.txt"))

#Rename first and second columns with activity number and activity name
activities <- rename(activities, activityNumber = V1, activityName = V2)

#Merge data set with activities using activity number as link
dt <- merge(data, activities, by="activityNumber", all.x=TRUE)

#END OF STEP 3

#STEP 4. Appropriately labels the data set with descriptive activity names

#Use features file for obtaining feature names
featureV <- features$"featureColumn"
featureName <- features$"featureName"

#Manipulate the names of featureName list

#Remove ()
featureName <- sub("()","",as.character(features$featureName),fixed=TRUE)
#Change mean for Mean
featureName <- sub("mean","Mean",featureName,fixed=TRUE)
#Change std for StdDev
featureName <- sub("std","StdDev",featureName,fixed=TRUE)
#Change BodyBody for Body
featureName <- sub("BodyBody","Body",featureName,fixed=TRUE)
#Change Acc for Acceleration
featureName <- sub("Acc","Acceleration",featureName,fixed=TRUE)
#Change Gyro for Orientation
featureName <- sub("Gyro","Orientation",featureName,fixed=TRUE)
#Change Mag for Magnitude
featureName <- sub("Mag","Magnitude",featureName,fixed=TRUE)
#Change tBody for timeBody
featureName <- sub("tBody","timeBody",featureName,fixed=TRUE)
#Change tGravity for timeGravity
featureName <- sub("tGravity","timeGravity",featureName,fixed=TRUE)
#Change fBody for freqBody
featureName <- sub("fBody","freqBody",featureName,fixed=TRUE)
#Change fGravity for freqGravity
featureName <- sub("fGravity","freqGravity",featureName,fixed=TRUE)

#Change column names in data set with featureName
for(i in 1:length(featureV)){
  names(dt)[names(dt)==featureV[i]] <- featureName[i]
}

#END OF STEP 4

#STEP 5. Creation of a tidy data set with the average of each variable for each activity 
#       and each subject

#Eliminate activityNumber column of dataset
dt <- select(dt, -activityNumber)

#Divide dataframe for each subject and activity, the value of each feature 
dt.melted <- melt(dt, id=c("subject","activityName"))

#Calculate the mean of each feature, for each subject and activity 
dt.mean <- dcast(dt.melted, subject + activityName ~ variable, mean)

#
write.table(dt.mean,"TDS.txt",row.names=FALSE,quote = FALSE)


