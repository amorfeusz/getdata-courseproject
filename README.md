This is essentially a copy of the run_analysis.R script where with expanded comments to further clarify/explain steps taken to generate the tidy dataset.

# 1. MERGE TRAINING AND TEST DATASETS TOGETHER                               
1. read in the 561 features or variable names
2. read in the activity labels
3. read in test data from the three separate files into three datasets, test_subject, test_X, and test_Y
4. read in training data (same logic as above)
5. combine train and test dataset files using the rbind() function.
6. now we cleanup memory by removing no longer needed datasets

# 3a & 4a Before Step 2, we perform processing to label the variable names and link the subjects and activities performed to the raw X dataset
1. label subject and y(activity) variable names, and for later the labels dataset for merging using the names() function.
2. cbind activity type to variables of data
3. cbind subjects to 562 variables of data
4. convert dataframe into data table so we can use the keying and merging functions (my preferred method)
5. clean up memory again
6. now we set the keys using setkeys() for the activity labels and the activityid fields and perform the merge.
7. more cleanup to keep environment tidy

# 4a continued, we prepare the variable names to make them fit tidy princinples
1. Process functions to clean up the variable names (avoiding capital letters, removing underscores, dashses, periods and the parantheses from the mean(), std(), etc. portions of variable names
2. cleanup
3. We have now completed a full merging of the train and test datasets along with tidy princinples cleaup and re-integration of key variables with the main data (X)

# 2. Now we extract only the measurements on the mean and standard deviation 
1. Convert data.table back into data.frame
2. identify which columns are mean() and create logical vector "mean"
3. identify which columns are std() and also create another logical vector
3. process the logical vectors to collapse them, create single vector which identifies the columns we wish to keep
4. extract only those mean and std measurements (this also removes the initial activity and subject variable columns)
5. also extract the activity and subject variables, then put the two new datasets back together
6. cleanup

# 4b. Appropriately labels the data set with descriptive variable names       
1. here we to some additional renaming to follow tidy variable naming principles (no excessive abbreviations, etc)
2. rename t, f, and gyro fragments to respective domain types for easier reading

# 5. Tidy dataset with average of each variable for each activity subject      #
1. Aggregate function used to calculate means of variables
2. because aggergate was applied to all columns, we must remove the non-sensical means of activity and subject columns. 
3. cleanup
4. output tidyX dataset using write.table