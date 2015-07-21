#Getting and Cleaning Data Course Project

https://class.coursera.org/getdata-030

Course Project

Human Activity Recognition Using Smartphones Data Set 

Calin Uioreanu 2015-07-21

###Tidy data definition

1. Each variable you measure should be in one column
2. Each different observation of that variable should be in a different row

###Tasks description

The goal of this project is to prepare tidy data that can be used for later analysis for the project Human Activity Recognition Using Smartphones Data Set. The requirements are to create one R script called run_analysis.R that does the following:

1. Merges the training and the test sets to create one data set.
2. Extracts only the measurements on the mean and standard deviation for each measurement. 
3. Uses descriptive activity names to name the activities in the data set
4. Appropriately labels the data set with descriptive variable names. 
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

### R Script

The R script is available here: [run_analysis.R](https://github.com/uioreanu/getdata-030-Course-Project/blob/master/run_analysis.R), it contains commented code grouped into modules for each section listed as tasks.

### Script description

A more detailed script description is available in the file: [CodeBook.md](https://github.com/uioreanu/getdata-030-Course-Project/blob/master/CodeBook.md), containing
a code book that describes the variables, the data, and any transformations or work that were performed to clean up the data

### Script Output

The R code outputs several metrics as it develops, and outputs at the very end the [tidySet.txt](https://github.com/uioreanu/getdata-030-Course-Project/blob/master/tidySet.txt) TEXT file, containing
a tidy set with mean values for Mean and Standard Deviation across the entire dataset, grouped by Individual Subject and Activitiy. The file is generated according to the Tidy Dataset principles.

