##CodeBook.md
author: Calin Uioreanu

date:   2015-07-21

###a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data

The raw data is loaded from the data directories, merged into one dataset, from this dataset the columns containing mean and standard deviation values are extracted, normalised and the result is written to a tidy dataset file.

First step, cleaning up the environment and check if the data directory is present

```r
#######################################
# cleanup & setup
rm(list=ls())
gc()
setwd('data_R/getdata-030/')

# libs - no libs, base R packages

# datadir
datadir = 'UCI HAR Dataset'
if (!file.exists(datadir)) {
  print("Please download the Human+Activity+Recognition+Using+Smartphones dataset from Coursera, unzip it then rerun.")
  stop()
}
#######################################
```

Second step is data feching 

```r
#######################################
# PART 0
# DATA FETCHING

# start the timer
ptm <- proc.time()

# read labels
activity_labels = read.table(paste0(datadir, "/activity_labels.txt"),    header = FALSE)
feature_labels  = read.table(paste0(datadir, "/features.txt"),           header = FALSE)

# load train raw data
subject_train   = read.table(paste0(datadir, "/train/subject_train.txt"),header = FALSE)
X_train         = read.table(paste0(datadir, "/train/X_train.txt"),      header = FALSE)
y_train         = read.table(paste0(datadir, "/train/y_train.txt"),      header = FALSE)

# load train raw data
subject_test    = read.table(paste0(datadir, "/test/subject_test.txt"),  header = FALSE)
X_test          = read.table(paste0(datadir, "/test/X_test.txt"),        header = FALSE)
y_test          = read.table(paste0(datadir, "/test/y_test.txt"),        header = FALSE)

# stop timer
proc.time()-ptm
# User      System verstrichen 
# 29.52        0.08       29.75 
#######################################

```

Part1 of the tasklist 

```r
#######################################
# PART 1
# Merge the training and the test sets to create one data set.

# DATA PREPARATION
# merge training with test set
subject  <- rbind(subject_train, subject_test)
activity <- rbind(y_train, y_test)
features <- rbind(X_train, X_test)

print(object.size(features), units="Mb")
#44.1 Mb

# extract proper column names 
colnames(subject)  <- "Subject"
colnames(activity) <- "Activity"

# syntactically valid feature names are handled in Part 4 
# f.e. angle(tBodyGyroJerkMean,gravityMean) => angle.tBodyGyroJerkMean.gravityMean.
# names(features)   <- make.names(feature_labels$V2, unique = TRUE)
names(features)   <- feature_labels$V2

# gc()
feature_labels <- NULL

# TIDY DATA BUILDING
dataSet <- cbind(subject, activity, features)

# checking metrics
dim(X_train)
# 7352  561
dim(X_test)
# 2947  561
dim(dataSet)
#[1] 10299   563
# 7352+2947 = 10299, 561 + subject & activity = 563
#######################################

```

Part2 of the tasklist 

```r

#######################################
# PART 2
# Extracts only the measurements on the mean and standard deviation for each measurement. 
# fetch features containing "mean"
meanColumns <- grep("mean", names(dataSet), value = TRUE, ignore.case = TRUE)

# fetch features containing "std" (standard deviation)
stdColumns  <- grep("std", names(dataSet), value = TRUE, ignore.case = TRUE)

# browse results
str(dataSet[,meanColumns])
# 'data.frame':	10299 obs. of  53 variables:
# $ tBodyAcc.mean...X                   : num  0.289 0.278 0.28 0.279 0.277 ...
# $ tBodyAcc.mean...Y                   : num  -0.0203 -0.0164 -0.0195 -0.0262 -0.0166 ...
# $ tBodyAcc.mean...Z                   : num  -0.133 -0.124 -0.113 -0.123 -0.115 ...
# $ tGravityAcc.mean...X                : num  0.963 0.967 0.967 0.968 0.968 ...

colMeans(dataSet[,meanColumns])
# tBodyAcc.mean...X                    tBodyAcc.mean...Y 
# 0.274347261                         -0.017743492 
# tBodyAcc.mean...Z                 tGravityAcc.mean...X 
# -0.108925033                          0.669226222 

str(dataSet[,stdColumns])
# 'data.frame':	10299 obs. of  33 variables:
# $ tBodyAcc.std...X          : num  -0.995 -0.998 -0.995 -0.996 -0.998 ...
# $ tBodyAcc.std...Y          : num  -0.983 -0.975 -0.967 -0.983 -0.981 ...
# $ tBodyAcc.std...Z          : num  -0.914 -0.96 -0.979 -0.991 -0.99 ...

str(dataSet[,c(meanColumns, stdColumns)])
# 'data.frame':	10299 obs. of  86 variables:
# $ tBodyAcc.mean...X                   : num  0.289 0.278 0.28 0.279 0.277 ...
# $ tBodyAcc.mean...Y                   : num  -0.0203 -0.0164 -0.0195 -0.0262 -0.0166 ...
# $ tBodyAcc.mean...Z                   : num  -0.133 -0.124 -0.113 -0.123 -0.115 ...

# extract subset with the two binded columns and only mean and std columns
subSet <- dataSet[,c('Subject', 'Activity', meanColumns, stdColumns)]
dim(subSet)
#[1] 10299    88
#######################################


```

Part3 of the tasklist 

```r


#######################################
# PART 3
# Uses descriptive activity names to name the activities in the data set
# 
# Which data set is meant here? process the main dataSet then reflect it on the subSet
# logic: for each activity, replace the activity with the TEXTual description
#

# current status evaluation
activity_labels
# V1                 V2
# 1  1            WALKING
# 2  2   WALKING_UPSTAIRS
# 3  3 WALKING_DOWNSTAIRS
# 4  4            SITTING
# 5  5           STANDING
# 6  6             LAYING
table(dataSet$Activity)
# 
# 1    2    3    4    5    6 
# 1722 1544 1406 1777 1906 1944 

dataSet$Activity <- as.character(dataSet$Activity)
for (i in 1:nrow(activity_labels)) {
  activityId  <- activity_labels$V1[i]
  # we need factor -> character conversion
  activityTXT <- as.character(activity_labels$V2[i])
  
  dataSet[dataSet$Activity==activityId,]$Activity <- activityTXT
}
table(dataSet$Activity)
# 
# LAYING            SITTING           STANDING            WALKING 
# 1944               1777               1906               1722 
# WALKING_DOWNSTAIRS   WALKING_UPSTAIRS 
# 1406               1544 

# extract subset with only mean and std columns
subSet <- dataSet[,c('Subject', 'Activity', meanColumns, stdColumns)]
dim(subSet)
#[1] 10299    88
#######################################


```

Part4 of the tasklist 

```r



#######################################
# PART 4
# Appropriately labels the data set with descriptive variable names. 
# 
(columnNames = names(subSet))
# [1] "Subject"                              "Activity"                            
# [3] "tBodyAcc-mean()-X"                    "tBodyAcc-mean()-Y"                   
# [5] "tBodyAcc-mean()-Z"                    "tGravityAcc-mean()-X"                
# [7] "tGravityAcc-mean()-Y"                 "tGravityAcc-mean()-Z"                
# [9] "tBodyAccJerk-mean()-X"                "tBodyAccJerk-mean()-Y"               
# [11] "tBodyAccJerk-mean()-Z"                "tBodyGyro-mean()-X"                  
# [13] "tBodyGyro-mean()-Y"                   "tBodyGyro-mean()-Z"                  
# [15] "tBodyGyroJerk-mean()-X"               "tBodyGyroJerk-mean()-Y"              
# [17] "tBodyGyroJerk-mean()-Z"               "tBodyAccMag-mean()"                  
# [19] "tGravityAccMag-mean()"                "tBodyAccJerkMag-mean()"              
# [21] "tBodyGyroMag-mean()"                  "tBodyGyroJerkMag-mean()"             
# [23] "fBodyAcc-mean()-X"                    "fBodyAcc-mean()-Y"                   
# [25] "fBodyAcc-mean()-Z"                    "fBodyAcc-meanFreq()-X"               
# [27] "fBodyAcc-meanFreq()-Y"                "fBodyAcc-meanFreq()-Z"               
# [29] "fBodyAccJerk-mean()-X"                "fBodyAccJerk-mean()-Y"               
# [31] "fBodyAccJerk-mean()-Z"                "fBodyAccJerk-meanFreq()-X"           
# [33] "fBodyAccJerk-meanFreq()-Y"            "fBodyAccJerk-meanFreq()-Z"           
# [35] "fBodyGyro-mean()-X"                   "fBodyGyro-mean()-Y"                  
# [37] "fBodyGyro-mean()-Z"                   "fBodyGyro-meanFreq()-X"              
# [39] "fBodyGyro-meanFreq()-Y"               "fBodyGyro-meanFreq()-Z"              
# [41] "fBodyAccMag-mean()"                   "fBodyAccMag-meanFreq()"              
# [43] "fBodyBodyAccJerkMag-mean()"           "fBodyBodyAccJerkMag-meanFreq()"      
# [45] "fBodyBodyGyroMag-mean()"              "fBodyBodyGyroMag-meanFreq()"         
# [47] "fBodyBodyGyroJerkMag-mean()"          "fBodyBodyGyroJerkMag-meanFreq()"     
# [49] "angle(tBodyAccMean,gravity)"          "angle(tBodyAccJerkMean),gravityMean)"
# [51] "angle(tBodyGyroMean,gravityMean)"     "angle(tBodyGyroJerkMean,gravityMean)"
# [53] "angle(X,gravityMean)"                 "angle(Y,gravityMean)"                
# [55] "angle(Z,gravityMean)"                 "tBodyAcc-std()-X"                    
# [57] "tBodyAcc-std()-Y"                     "tBodyAcc-std()-Z"                    
# [59] "tGravityAcc-std()-X"                  "tGravityAcc-std()-Y"                 
# [61] "tGravityAcc-std()-Z"                  "tBodyAccJerk-std()-X"                
# [63] "tBodyAccJerk-std()-Y"                 "tBodyAccJerk-std()-Z"                
# [65] "tBodyGyro-std()-X"                    "tBodyGyro-std()-Y"                   
# [67] "tBodyGyro-std()-Z"                    "tBodyGyroJerk-std()-X"               
# [69] "tBodyGyroJerk-std()-Y"                "tBodyGyroJerk-std()-Z"               
# [71] "tBodyAccMag-std()"                    "tGravityAccMag-std()"                
# [73] "tBodyAccJerkMag-std()"                "tBodyGyroMag-std()"                  
# [75] "tBodyGyroJerkMag-std()"               "fBodyAcc-std()-X"                    
# [77] "fBodyAcc-std()-Y"                     "fBodyAcc-std()-Z"                    
# [79] "fBodyAccJerk-std()-X"                 "fBodyAccJerk-std()-Y"                
# [81] "fBodyAccJerk-std()-Z"                 "fBodyGyro-std()-X"                   
# [83] "fBodyGyro-std()-Y"                    "fBodyGyro-std()-Z"                   
# [85] "fBodyAccMag-std()"                    "fBodyBodyAccJerkMag-std()"           
# [87] "fBodyBodyGyroMag-std()"               "fBodyBodyGyroJerkMag-std()"  

columnNames<- sub(pattern="BodyBody", replacement="Body", x=columnNames)
columnNames<- sub(pattern="Gyro", replacement="Gyroscope", x=columnNames)
columnNames<- sub(pattern="Acc", replacement="Acceleration", x=columnNames)
columnNames<- sub(pattern="Mag", replacement="Magnitude", x=columnNames)
columnNames<- sub(pattern="-mean\\(\\)", replacement="Mean", x=columnNames)
columnNames<- sub(pattern="-meanFreq\\(\\)", replacement="MeanFreq", x=columnNames)
columnNames<- sub(pattern="-std\\(\\)", replacement="Std", x=columnNames)
columnNames<- sub(pattern="-X$", replacement="X", x=columnNames)
columnNames<- sub(pattern="-Y$", replacement="Y", x=columnNames)
columnNames<- sub(pattern="-Z$", replacement="Z", x=columnNames)
columnNames<- sub(pattern="^t", replacement="Time", x=columnNames)
columnNames<- sub(pattern="^f", replacement="Frequency", x=columnNames)
# replace remaining patterns
columnNames<- make.names(columnNames)
columnNames
# [1] "Subject"                                        "Activity"                                       "TimeBodyAccelerationMeanX"                     
# [4] "TimeBodyAccelerationMeanY"                      "TimeBodyAccelerationMeanZ"                      "TimeGravityAccelerationMeanX"                  
# [7] "TimeGravityAccelerationMeanY"                   "TimeGravityAccelerationMeanZ"                   "TimeBodyAccelerationJerkMeanX"                 
# [10] "TimeBodyAccelerationJerkMeanY"                  "TimeBodyAccelerationJerkMeanZ"                  "TimeBodyGyroscopeMeanX"                        
# [13] "TimeBodyGyroscopeMeanY"                         "TimeBodyGyroscopeMeanZ"                         "TimeBodyGyroscopeJerkMeanX"                    
# [16] "TimeBodyGyroscopeJerkMeanY"                     "TimeBodyGyroscopeJerkMeanZ"                     "TimeBodyAccelerationMagnitudeMean"             
# [19] "TimeGravityAccelerationMagnitudeMean"           "TimeBodyAccelerationJerkMagnitudeMean"          "TimeBodyGyroscopeMagnitudeMean"                
# [22] "TimeBodyGyroscopeJerkMagnitudeMean"             "FrequencyBodyAccelerationMeanX"                 "FrequencyBodyAccelerationMeanY"                
# [25] "FrequencyBodyAccelerationMeanZ"                 "FrequencyBodyAccelerationMeanFreqX"             "FrequencyBodyAccelerationMeanFreqY"            
# [28] "FrequencyBodyAccelerationMeanFreqZ"             "FrequencyBodyAccelerationJerkMeanX"             "FrequencyBodyAccelerationJerkMeanY"            
# [31] "FrequencyBodyAccelerationJerkMeanZ"             "FrequencyBodyAccelerationJerkMeanFreqX"         "FrequencyBodyAccelerationJerkMeanFreqY"        
# [34] "FrequencyBodyAccelerationJerkMeanFreqZ"         "FrequencyBodyGyroscopeMeanX"                    "FrequencyBodyGyroscopeMeanY"                   
# [37] "FrequencyBodyGyroscopeMeanZ"                    "FrequencyBodyGyroscopeMeanFreqX"                "FrequencyBodyGyroscopeMeanFreqY"               
# [40] "FrequencyBodyGyroscopeMeanFreqZ"                "FrequencyBodyAccelerationMagnitudeMean"         "FrequencyBodyAccelerationMagnitudeMeanFreq"    
# [43] "FrequencyBodyAccelerationJerkMagnitudeMean"     "FrequencyBodyAccelerationJerkMagnitudeMeanFreq" "FrequencyBodyGyroscopeMagnitudeMean"           
# [46] "FrequencyBodyGyroscopeMagnitudeMeanFreq"        "FrequencyBodyGyroscopeJerkMagnitudeMean"        "FrequencyBodyGyroscopeJerkMagnitudeMeanFreq"   
# [49] "angle.tBodyAccelerationMean.gravity."           "angle.tBodyAccelerationJerkMean..gravityMean."  "angle.tBodyGyroscopeMean.gravityMean."         
# [52] "angle.tBodyGyroscopeJerkMean.gravityMean."      "angle.X.gravityMean."                           "angle.Y.gravityMean."                          
# [55] "angle.Z.gravityMean."                           "TimeBodyAccelerationStdX"                       "TimeBodyAccelerationStdY"                      
# [58] "TimeBodyAccelerationStdZ"                       "TimeGravityAccelerationStdX"                    "TimeGravityAccelerationStdY"                   
# [61] "TimeGravityAccelerationStdZ"                    "TimeBodyAccelerationJerkStdX"                   "TimeBodyAccelerationJerkStdY"                  
# [64] "TimeBodyAccelerationJerkStdZ"                   "TimeBodyGyroscopeStdX"                          "TimeBodyGyroscopeStdY"                         
# [67] "TimeBodyGyroscopeStdZ"                          "TimeBodyGyroscopeJerkStdX"                      "TimeBodyGyroscopeJerkStdY"                     
# [70] "TimeBodyGyroscopeJerkStdZ"                      "TimeBodyAccelerationMagnitudeStd"               "TimeGravityAccelerationMagnitudeStd"           
# [73] "TimeBodyAccelerationJerkMagnitudeStd"           "TimeBodyGyroscopeMagnitudeStd"                  "TimeBodyGyroscopeJerkMagnitudeStd"             
# [76] "FrequencyBodyAccelerationStdX"                  "FrequencyBodyAccelerationStdY"                  "FrequencyBodyAccelerationStdZ"                 
# [79] "FrequencyBodyAccelerationJerkStdX"              "FrequencyBodyAccelerationJerkStdY"              "FrequencyBodyAccelerationJerkStdZ"             
# [82] "FrequencyBodyGyroscopeStdX"                     "FrequencyBodyGyroscopeStdY"                     "FrequencyBodyGyroscopeStdZ"                    
# [85] "FrequencyBodyAccelerationMagnitudeStd"          "FrequencyBodyAccelerationJerkMagnitudeStd"      "FrequencyBodyGyroscopeMagnitudeStd"            
# [88] "FrequencyBodyGyroscopeJerkMagnitudeStd"

length(names(subSet))
length(columnNames)
#[1] 88
names(subSet) <- columnNames
#######################################


```

Part5 of the tasklist 

```r

#######################################
# PART 5
# From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
# 
table(subSet$Subject,subSet$Activity)
# 
# LAYING SITTING STANDING WALKING WALKING_DOWNSTAIRS WALKING_UPSTAIRS
# 1      50      47       53      95                 49               53
# 2      48      46       54      59                 47               48
# 3      62      52       61      58                 49               59
# 4      54      50       56      60                 45               52
# ...

colMeansStd <- names(subSet)
length(colMeansStd)
#[1] 88
colMeansStd <- colMeansStd[!(names(subSet) %in% c('Subject', 'Activity'))]
length(colMeansStd)
#[1] 86

#the average of each variable for each activity and each subject.
tidySet <- aggregate(. ~ Subject + Activity, subSet, mean)

dim(tidySet)
#[1] 180  88

tidySet[1,]
# Subject Activity TimeBodyAccelerationMeanX TimeBodyAccelerationMeanY TimeBodyAccelerationMeanZ TimeGravityAccelerationMeanX
# 1       1   LAYING                 0.2215982               -0.04051395                -0.1132036                   -0.2488818
# TimeGravityAccelerationMeanY TimeGravityAccelerationMeanZ TimeBodyAccelerationJerkMeanX TimeBodyAccelerationJerkMeanY TimeBodyAccelerationJerkMeanZ
# 1                    0.7055498                    0.4458177                    0.08108653                   0.003838204                    0.01083424
# TimeBodyGyroscopeMeanX TimeBodyGyroscopeMeanY TimeBodyGyroscopeMeanZ TimeBodyGyroscopeJerkMeanX TimeBodyGyroscopeJerkMeanY
# 1            -0.01655309            -0.06448612              0.1486894                 -0.1072709                -0.04151729
# TimeBodyGyroscopeJerkMeanZ TimeBodyAccelerationMagnitudeMean TimeGravityAccelerationMagnitudeMean TimeBodyAccelerationJerkMagnitudeMean
# 1                -0.07405012                        -0.8419292                           -0.8419292                            -0.9543963
# TimeBodyGyroscopeMagnitudeMean TimeBodyGyroscopeJerkMagnitudeMean FrequencyBodyAccelerationMeanX FrequencyBodyAccelerationMeanY
# 1                     -0.8747595                          -0.963461                     -0.9390991                     -0.8670652
# FrequencyBodyAccelerationMeanZ FrequencyBodyAccelerationMeanFreqX FrequencyBodyAccelerationMeanFreqY FrequencyBodyAccelerationMeanFreqZ
# 1                     -0.8826669                         -0.1587927                         0.09753484                         0.08943766
# FrequencyBodyAccelerationJerkMeanX FrequencyBodyAccelerationJerkMeanY FrequencyBodyAccelerationJerkMeanZ FrequencyBodyAccelerationJerkMeanFreqX
# 1                         -0.9570739                         -0.9224626                         -0.9480609                              0.1324191
# FrequencyBodyAccelerationJerkMeanFreqY FrequencyBodyAccelerationJerkMeanFreqZ FrequencyBodyGyroscopeMeanX FrequencyBodyGyroscopeMeanY
# 1                             0.02451362                             0.02438795                  -0.8502492                  -0.9521915
# FrequencyBodyGyroscopeMeanZ FrequencyBodyGyroscopeMeanFreqX FrequencyBodyGyroscopeMeanFreqY FrequencyBodyGyroscopeMeanFreqZ
# 1                  -0.9093027                    -0.003546796                     -0.09152913                      0.01045813
# FrequencyBodyAccelerationMagnitudeMean FrequencyBodyAccelerationMagnitudeMeanFreq FrequencyBodyAccelerationJerkMagnitudeMean
# 1                             -0.8617676                                 0.08640856                                 -0.9333004
# FrequencyBodyAccelerationJerkMagnitudeMeanFreq FrequencyBodyGyroscopeMagnitudeMean FrequencyBodyGyroscopeMagnitudeMeanFreq
# 1                                      0.2663912                          -0.8621902                               -0.139775
# FrequencyBodyGyroscopeJerkMagnitudeMean FrequencyBodyGyroscopeJerkMagnitudeMeanFreq angle.tBodyAccelerationMean.gravity.
# 1                              -0.9423669                                   0.1764859                           0.02136597
# angle.tBodyAccelerationJerkMean..gravityMean. angle.tBodyGyroscopeMean.gravityMean. angle.tBodyGyroscopeJerkMean.gravityMean. angle.X.gravityMean.
# 1                                   0.003060407                          -0.001666985                                0.08443716            0.4267062
# angle.Y.gravityMean. angle.Z.gravityMean. TimeBodyAccelerationStdX TimeBodyAccelerationStdY TimeBodyAccelerationStdZ TimeGravityAccelerationStdX
# 1           -0.5203438           -0.3524131               -0.9280565               -0.8368274               -0.8260614                    -0.89683
# TimeGravityAccelerationStdY TimeGravityAccelerationStdZ TimeBodyAccelerationJerkStdX TimeBodyAccelerationJerkStdY TimeBodyAccelerationJerkStdZ
# 1                    -0.90772                  -0.8523663                   -0.9584821                   -0.9241493                   -0.9548551
# TimeBodyGyroscopeStdX TimeBodyGyroscopeStdY TimeBodyGyroscopeStdZ TimeBodyGyroscopeJerkStdX TimeBodyGyroscopeJerkStdY TimeBodyGyroscopeJerkStdZ
# 1            -0.8735439            -0.9510904            -0.9082847                -0.9186085                -0.9679072                -0.9577902
# TimeBodyAccelerationMagnitudeStd TimeGravityAccelerationMagnitudeStd TimeBodyAccelerationJerkMagnitudeStd TimeBodyGyroscopeMagnitudeStd
# 1                       -0.7951449                          -0.7951449                           -0.9282456                    -0.8190102
# TimeBodyGyroscopeJerkMagnitudeStd FrequencyBodyAccelerationStdX FrequencyBodyAccelerationStdY FrequencyBodyAccelerationStdZ
# 1                         -0.935841                    -0.9244374                    -0.8336256                    -0.8128916
# FrequencyBodyAccelerationJerkStdX FrequencyBodyAccelerationJerkStdY FrequencyBodyAccelerationJerkStdZ FrequencyBodyGyroscopeStdX
# 1                        -0.9641607                        -0.9322179                         -0.960587                 -0.8822965
# FrequencyBodyGyroscopeStdY FrequencyBodyGyroscopeStdZ FrequencyBodyAccelerationMagnitudeStd FrequencyBodyAccelerationJerkMagnitudeStd
# 1                  -0.951232                 -0.9165825                            -0.7983009                                 -0.921804
# FrequencyBodyGyroscopeMagnitudeStd FrequencyBodyGyroscopeJerkMagnitudeStd
# 1                         -0.8243194                             -0.9326607

# Please upload the tidy data set created in step 5 of the instructions. Please upload your data set as a txt file created with write.table() using row.name=FALSE (do not cut and paste a dataset directly into the text box, as this may cause errors saving your submission).
# 
# row.name=FALSE   WRONG
# row.names=FALSE  OK
#
write.table(file = 'tidySet.txt', tidySet, row.names=FALSE)
#######################################

```
