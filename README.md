################################################################################
### 1. MERGE TRAINING AND TEST DATASETS TOGETHER                               #
################################################################################

#read in the 561 features or variable names
features<-read.delim('UCI HAR Dataset\\features.txt',header=F, sep=" ")

#read in the activity labels
labels<- read.delim('UCI HAR Dataset\\activity_labels.txt',header=F, sep=" ")

#read in test data
test_subject<- read.delim('UCI HAR Dataset\\test\\subject_test.txt',header=F, sep=" ")
test_X<- read.table('UCI HAR Dataset\\test\\X_test.txt',header=F, sep = "")
test_y<- read.delim('UCI HAR Dataset\\test\\y_test.txt',header=F, sep=" ")

#read in training data
train_subject<- read.delim('UCI HAR Dataset\\train\\subject_train.txt',header=F, sep=" ")
train_X<- read.table('UCI HAR Dataset\\train\\X_train.txt',header=F, sep = "")
train_y<- read.delim('UCI HAR Dataset\\train\\y_train.txt',header=F, sep=" ")

#combine train and test dataset files
subject<-rbind(test_subject,train_subject)
X<- rbind(test_X,train_X)
y<- rbind(test_y,train_y)

#clear test and train from memory to clear workspace
rm(test_X, test_y, test_subject, train_X, train_y, train_subject)

#label subject and y(activity) variable names, and for later the labels dataset
#for merging
names(subject)<-"subject"
names(y)<-"activityid"
names(labels)<-c("activityid", "activity")

#bind activity type to variables of data
X2 <- cbind(y,X)
#bind subjects to 562 variables of data
X3 <- cbind(subject,X2)

#load data table library and convert data frame into data table
library(data.table)
X4<-data.table(X3)
labels2<-data.table(labels)

#cleanup
rm(X,X2,X3,subject,y,labels)

#set keys and merge activity and labels - STEP 3 of assignment done early
setkey(labels2, activityid)
setkey(X4, activityid)
X5<-merge(labels2, X4)
#cleanup
rm(X4, labels2)

##prepare variable names

#tidy up variable names
features1<-tolower(features[,2])

features2<-c("activityid","activity", "subject")
features3<-c(features2, features1)

#further variable name cleanup
features3<-sub("-","", features3,)
features3<-sub("\\(","", features3,)
features3<-sub("\\)","", features3,)

#transfer variable names from features
names(X5)<-features3

#cleanup
rm(features, features1, features2)
#COMPLETED MERGING OF TRAIN AND TEST DATASETS

################################################################################
### 2. Extracts only the measurements on the mean and standard deviation ement #
################################################################################

#Convert data.table back into data.frame
X6<-data.frame(X5)

#identify which columns are mean
mean<- features3 %like% "mean"
#identify which columns are std
std<- features3 %like% "std"

newvar <- cbind(mean, std)
newvar2 <- newvar[,1]== TRUE |  newvar[,2] == TRUE

#extract
tidyX <- X6[,newvar2]
#extract initial variables
X7 <- X6[,2:3]
tidyX2 <- cbind(X7, tidyX)

#cleanup
rm(X6,X7,tidyX,X5, newvar, newvar2, mean, std,features3)

################################################################################
# 4. Appropriately labels the data set with descriptive variable names         #
################################################################################

# rename t, f, and gyro to respective domain types for easier reading
names(tidyX2)<-sub("tbody","timebody",names(tidyX2))
names(tidyX2)<-sub("tgravity","timegravity",names(tidyX2))

names(tidyX2)<-sub("fbody","frequencybody",names(tidyX2))
names(tidyX2)<-sub("fgravity","frequencygravity",names(tidyX2))

names(tidyX2)<-sub("gyro","gyroscope",names(tidyX2))
names(tidyX2)<-sub("\\.","",names(tidyX2))

################################################################################
# 5. Tidy dataset with average of each variable for each activity subject      #
################################################################################

#Aggregate function used to calculate means of variables
tidyX3<-aggregate(tidyX2, list(subject = tidyX2$subject, activity=tidyX2$activity), mean)
#because aggergate was applied to all columns, we must remove the non-sensical means of activity and subject columns. 
tidyX4<-tidyX3[,1:2]
tidyX5<-tidyX3[5:90]
tidyX<-cbind(tidyX4,tidyX5)
#cleanup
rm(tidyX2, tidyX3, tidyX4, tidyX5)

#output tidyX dataset
write.table(tidyX,"tidyX.txt", row.names=FALSE)
