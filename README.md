This project imports and cleans data as specified. The data we are using
for this project is a Human Activity Recognition project using
Smartphone data. The data is stored in /getdata\_projectfiles\_UCI HAR
Dataset/UCI HAR Dataset/. We will gather this data and clean it in
“run\_analysis.R”. We walk through that code below.

A codebook has been generated for each final data frame (two in total)
using the datamaid library.

    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

Import
======

    # Import #
    data_dir = "./getdata_projectfiles_UCI HAR Dataset/UCI HAR Dataset/"
    features <- read.table(paste0(data_dir, "features.txt"), col.names = c("row", "functions"))
    activity_labels <- read.table(paste0(data_dir, "activity_labels.txt"), col.names = c("class", "activity"))

    test_data_dir = "./getdata_projectfiles_UCI HAR Dataset/UCI HAR Dataset/test/"
    subject_test <- read.table(paste0(test_data_dir, "subject_test.txt"), col.names = "subject")
    x_test <- read.table(paste0(test_data_dir, "X_test.txt"), col.names = features$functions)
    y_test <- read.table(paste0(test_data_dir, "y_test.txt"), col.names = "class")

    train_data_dir = "./getdata_projectfiles_UCI HAR Dataset/UCI HAR Dataset/train/"
    subject_train <- read.table(paste0(train_data_dir, "subject_train.txt"), col.names = "subject")
    x_train <- read.table(paste0(train_data_dir, "X_train.txt"), col.names = features$functions)
    y_train <- read.table(paste0(train_data_dir, "y_train.txt"), col.names = "class")

Tidy
====

1) Merge the training and the test sets to create one data set.
---------------------------------------------------------------

    x_df <- rbind(x_test, x_train)
    y_df <- rbind(y_test, y_train)
    subject_df <- rbind(subject_test, subject_train)

    HAR_df <- cbind(x_df, y_df, subject_df)

2) Extract only the measurements on the mean and standard deviation for each measurement.
-----------------------------------------------------------------------------------------

    HAR_tidy_df <- HAR_df %>%
      select(subject, class, contains("mean"), contains("std"))

3) Use descriptive activity names to name the activities in the data set.
-------------------------------------------------------------------------

    HAR_tidy_df <- inner_join(HAR_tidy_df, activity_labels, by = "class") %>%
      select(-class) %>%  # Drop class var
      relocate(subject, activity)  # Reorder vars to front

4) Appropriately label the data set with descriptive variable names.
--------------------------------------------------------------------

    names(HAR_tidy_df) <- gsub("Acc", "Accelerometer", names(HAR_tidy_df))
    names(HAR_tidy_df) <- gsub("Gyro", "Gyroscope", names(HAR_tidy_df))
    names(HAR_tidy_df) <- gsub("BodyBody", "Body", names(HAR_tidy_df))
    names(HAR_tidy_df)<-gsub("Mag", "Magnitude", names(HAR_tidy_df))
    names(HAR_tidy_df)<-gsub("^t", "Time", names(HAR_tidy_df))
    names(HAR_tidy_df)<-gsub("^f", "Frequency", names(HAR_tidy_df))
    names(HAR_tidy_df)<-gsub("tBody", "TimeBody", names(HAR_tidy_df))

5) From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
-------------------------------------------------------------------------------------------------------------------------------------------------

    HAR_agg_df <- HAR_tidy_df %>% 
      group_by(activity, subject) %>%
      summarize_all(funs(mean))

    ## Warning: `funs()` was deprecated in dplyr 0.8.0.
    ## Please use a list of either functions or lambdas: 
    ## 
    ##   # Simple named list: 
    ##   list(mean = mean, median = median)
    ## 
    ##   # Auto named with `tibble::lst()`: 
    ##   tibble::lst(mean, median)
    ## 
    ##   # Using lambdas
    ##   list(~ mean(., trim = .2), ~ median(., na.rm = TRUE))
