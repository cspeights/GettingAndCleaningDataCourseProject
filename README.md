# "Getting And Cleaning Data Course Project"

Instructions:
1   Merges the training and the test sets to create one data set.
2   Extracts only the measurements on the mean and standard deviation for each measurement. 
3   Uses descriptive activity names to name the activities in the data set
4   Appropriately labels the data set with descriptive variable names. 
5 From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.

```{r load, cache=TRUE}
library(dplyr)
```

First step: combine the training and test data

Set the working directory to training data folder and download the training data.

```{r train, cache=TRUE }
setwd("~\\JobResources\\R\\GettingandCleaningData\\GettingAndCleaningDataProject\\UCI HAR Dataset\\train")

xTrain <- read.table("X_train.txt")
yTrain <- read.table("y_train.txt")
trainSubjects <- read.table("subject_train.txt")
```

Set the working directory to test data folder and download the test data

```{r testdatadownload, cache = TRUE}
setwd("~\\JobResources\\R\\GettingandCleaningData\\GettingAndCleaningDataProject\\UCI HAR Dataset\\test")

xTest <- read.table("X_test.txt")
yTest <- read.table("y_test.txt")
testSubjects <- read.table("subject_test.txt")
```

Get the labels.
```{r, cache=TRUE}
setwd("~\\JobResources\\R\\GettingandCleaningData\\GettingAndCleaningDataProject\\UCI HAR Dataset")
dataNames <- read.table("features.txt")
```

Combine test and training data
```{r combine, cache=TRUE}
xData <- rbind(xTest, xTrain)
yData <- rbind(yTest, yTrain)
subjectData <- rbind(testSubjects, trainSubjects)

#Combine both X and Y data in the same table
data <- cbind(xData, yData, subjectData)

#Add names
names(data) <- dataNames$V2
```


Step two: Select the mean and standard deviation columns
```{r selectColumns, cache=TRUE}
meanStdevDataCol <- grep("(mean|std)\\(\\)",colnames(data))
meanStdevData <- data[,c(meanStdevDataCol,562:563)]

names(meanStdevData) <- c(names(meanStdevData[1:66]), "activity", "subject")
```

Step three: Rename the activities with human-readable names
```{r readable, cache=TRUE}
Activity <- factor(meanStdevData$activity, labels = c("WALKING","WALKING_UPSTAIRS","WALKING_DOWNSTAIRS","SITTING","STANDING","LAYING"))
tidyData <- cbind(meanStdevData, Activity)
tidyData <- select(tidyData, -activity)
```

Step four: Make the column names tidy
```{r cleanNames, cache=TRUE}
tidyNames1 <- gsub("tBody", "timeBody", names(tidyData))
tidyNames2 <- gsub("fBody", "freqBody", tidyNames1)
tidyNames3 <- gsub("tGravity", "timeGravity", tidyNames2)
tidyNames4 <- gsub("\\()","",tidyNames3)
tidyNames5 <- gsub("\\-","",tidyNames4)
tidyNames6 <- gsub("mean","Mean",tidyNames5)
tidyNames  <- gsub("std","Std",tidyNames6)

names(tidyData) <- tidyNames

tidyData <- select(tidyData, Activity, subject, timeBodyAccMeanX:freqBodyBodyGyroJerkMagStd)
```


Step five: From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
```{r finalStep, cache=TRUE, warning=FALSE}
finalData <- aggregate(tidyData, by = list(tidyData$subject, Activity), FUN = mean)

finalTidyData <- select(finalData, Group.2, Group.1, timeBodyAccMeanX:freqBodyBodyGyroJerkMagStd)

names(finalTidyData) <- names(tidyData)

write.table(finalTidyData, file = "FinalData.txt", row.name = FALSE)     
```

