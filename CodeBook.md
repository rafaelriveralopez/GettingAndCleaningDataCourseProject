#Description of raw data

The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data.

The sensor signals (accelerometer and gyroscope) were pre-processed by applying noise filters and then sampled in fixed-width sliding windows of 2.56 sec and 50% overlap (128 readings/window). The sensor acceleration signal, which has gravitational and body motion components, was separated using a Butterworth low-pass filter into body acceleration and gravity. The gravitational force is assumed to have only low frequency components, therefore a filter with 0.3 Hz cutoff frequency was used. From each window, a vector of features was obtained by calculating variables from the time and frequency domain.

Attribute Information:

For each record in the dataset it is provided:
- Triaxial acceleration from the accelerometer (total acceleration) and the estimated body acceleration.
- Triaxial Angular velocity from the gyroscope.
- A 561-feature vector with time and frequency domain variables.
- Its activity label.
- An identifier of the subject who carried out the experiment. 

#Description of variables in tidy dataset
The tidy data set has 180 obs. of  68 variables

Varible | Description
------------ | -------------
subject | ID the subject who performed the activity for each window sample. Its range is from 1 to 30.
activityName |  WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING


timeBodyAcceleration |Mean and Standard deviation in X,Y,Z and total.  
timeGravityAcceleration | Mean and Standard deviation in X,Y,Z and total
timeBodyAccelerationJerk| Mean and Standard deviation in X,Y,Z and total
timeBodyOrientation | Mean and Standard deviation in X,Y,Z and total
timeBodyOrientationJerk | Mean and Standard deviation in X,Y,Z and total
freqBodyAcceleration | Mean and Standard deviation in X,Y,Z and total 
freqBodyAccelerationJerk | Mean and Standard deviation in X,Y,Z and total
freqBodyOrientation | Mean and Standard deviation in X,Y,Z and total
freqBodyOrientationJerk | Mean and Standard deviation in X,Y,Z and total


#Description of Transformations

##Libraries used

* data.table
* dplyr

##STEP 1. Merge the training and the test sets to create one data set

Using data frame for storing files

```R

subjectTrain <- read.table(paste0(pathds,"train/subject_train.txt"))
subjectTest <- read.table(paste0(pathds,"test/subject_test.txt"))

xTrain <- read.table(paste0(pathds,"train/X_train.txt"))
yTrain <- read.table(paste0(pathds,"train/y_train.txt"))

xTest <- read.table(paste0(pathds,"test/X_test.txt"))
yTest <- read.table(paste0(pathds,"test/y_test.txt"))
```

Merge training and test data

```R

subject <- rbind(subjectTrain,subjectTest)
x <- rbind(xTrain,xTest)
y <- rbind(yTrain,yTest)
```

Rename the column name of subject data 

Rename the column name of y data (activity label) 

Merge columns of data

```R
subject <- rename(subject, subject = V1)
y <- rename(y, activityNumber = V1)
data <- cbind(x,subject,y)
```

##STEP 2. Extract only the measurements on the mean and standard deviation for each measurement

Read feature description file
```R
features <- read.table(paste(pathds,"features.txt",sep=""))
```
Rename columns V1 and V2 in features data frame
```R
features <- rename(features, featureNumber = V1, featureName=V2)
```
Identify the feature names that contain "mean" or "std"
```R
match<-grep("mean\\(\\)|std\\(\\)",features$"featureName")
```
Select the rows that feature number is matching
```R
features <- filter(features,features$"featureNumber" %in% match)
```
Add column with column name in data
```R 
features <- mutate(features,featureColumn = paste0("V",featureNumber))
```
Select only columns of data that are in features with mean or std
```R
select <- select(data,features$"featureNumber")
```
Add subject and activity number for data set
```R
data <- mutate(subject = data$"subject", activityNumber= data$"activityNumber", select)
```

##STEP 3. Uses descriptive activity names to name the activities in the data set.

Read activity labels file

Rename first and second columns with activity number and activity name

Merge data set with activities using activity number as link

```R
activities <- read.table(paste0(pathds,"activity_labels.txt"))
activities <- rename(activities, activityNumber = V1, activityName = V2)
dt <- merge(data, activities, by="activityNumber", all.x=TRUE)
```

##STEP 4. Appropriately labels the data set with descriptive activity names

Use features file for obtaining feature names
```R
featureV <- features$"featureColumn"
featureName <- features$"featureName"
```
Manipulate the names of featureName list

Remove ()
Change 
- mean for Mean
- std for StdDev
- BodyBody for Body
- Acc for Acceleration
- Gyro for Orientation
- Mag for Magnitude
- tBody for timeBody
- tGravity for timeGravity
- fBody for freqBody
- fGravity for freqGravity
```R
featureName <- sub("()","",as.character(features$featureName),fixed=TRUE)
featureName <- sub("mean","Mean",featureName,fixed=TRUE)
featureName <- sub("std","StdDev",featureName,fixed=TRUE)
featureName <- sub("BodyBody","Body",featureName,fixed=TRUE)
featureName <- sub("Acc","Acceleration",featureName,fixed=TRUE)
featureName <- sub("Gyro","Orientation",featureName,fixed=TRUE)
featureName <- sub("Mag","Magnitude",featureName,fixed=TRUE)
featureName <- sub("tBody","timeBody",featureName,fixed=TRUE)
featureName <- sub("tGravity","timeGravity",featureName,fixed=TRUE)
featureName <- sub("fBody","freqBody",featureName,fixed=TRUE)
featureName <- sub("fGravity","freqGravity",featureName,fixed=TRUE)
```
Change column names in data set with featureName
```R
for(i in 1:length(featureV)){
  names(dt)[names(dt)==featureV[i]] <- featureName[i]
}
```

##STEP 5. Creation of a tidy data set with the average of each variable for each activity and each subject

Eliminate activityNumber column of dataset
```R
dt <- select(dt, -activityNumber)
```
Divide dataframe for each subject and activity, the value of each feature 
```R
dt.melted <- melt(dt, id=c("subject","activityName"))
```

Calculate the mean of each feature, for each subject and activity 
```R
dt.mean <- dcast(dt.melted, subject + activityName ~ variable, mean)
```
Create the tidy data set
```R
write.table(dt.mean,"TDS.txt",row.names=FALSE,quote = FALSE)
```

