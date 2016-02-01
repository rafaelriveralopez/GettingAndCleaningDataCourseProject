# Getting and Cleaning Data - Course Project


This repository hosts the R code and documentation files for the Course Project of Coursera Course "Getting and Cleaning Data".

The dataset being used is: [Human Activity Recognition Using Smartphones](http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones)

##Files

The code takes for granted all the data is present in the same folder, un-compressed and without names altered.

`CodeBook.md` describes the variables, the data, and transformations that was performed to clean up the data.

`run_analysis.R` contains all the code developed for executing the next steps:

- Merges the training and the test sets and to create one data set.
- Extracts only the measurements on the mean and standard deviation for each measurement.
- Uses descriptive activity names to name the activities in the data set.
- Assigns appropriately labels the data set with descriptive variable names.
- Creates a tidy data set with the average of each variable for each activity and each subject.

The output of the 5th step is called averages_data.txt, and uploaded in the course project's form.
