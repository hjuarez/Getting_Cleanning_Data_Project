# Getting and Cleanning Data course project
### Autor: Héctor Juárez Lara
###### Date: April 23rd, 2015

##### The next code prepare tidy data that can be used for later analysis from all measures saved from sensors that register lots of info about a man activity that wears the sensors.

###### Usage: Download and unzip the file "WD_data.zip" in your working dir, load in r studio the scritp: "run_analysis.R and execute it. After finish the execution you'll find de result "tidyData.txt" a text file with the tidy data with coma separated fields (variables)

##### Load Libraries requiered
```{r}
library(plyr)
library(reshape2)
```

##### Set the working directory. Must adjusted with local path for the first run.
```{r}
#setwd(file.path(getwd(),"G_Clean_Data"))
```
##### Read Training files and features and activity labels
```{r}
featuresfile <- "./data/features.txt"
Xtrainfile <- "./data/X_train.txt"
ytrainfile <- "./data/y_train.txt"
subTrainfile <- "./data/subject_train.txt"
activityLabs <- "./data/activity_labels.txt"
features <- read.table(featuresfile, header=F, sep=" ")
X_train <- read.table(Xtrainfile, header=F)
y_train <- read.table(ytrainfile, header=F)
subTrain <- read.table(subTrainfile, header=F)
ActLabels <- read.table(activityLabs, header=F)
```

##### Read Test files
```{r}
Xtestfile <- "./data/X_test.txt"
ytestfile <- "./data/y_test.txt"
subTestfile <- "./data/subject_test.txt"
X_test <- read.table(Xtestfile, header=F)
y_test <- read.table(ytestfile, header=F)
subTest <- read.table(subTestfile, header=F)
```

##### Asign features to column's names
```{r}
colnames(X_train) <- features$V2
colnames(X_test) <- features$V2
```

##### Join al measurements with subject in study and the activity
```{r}
df_Train <- cbind(subTrain,y_train,X_train)
colnames(df_Train)[1] <- "Subject_id"
colnames(df_Train)[2] <- "Activity_id"

df_Test <- cbind(subTest,y_test,X_test)
colnames(df_Test)[1] <- "Subject_id"
colnames(df_Test)[2] <- "Activity_id"
```

##### Incluye activity labels
```{r}
df_Train$Activity_Desc <- ActLabels$V2[df_Train$Activity_id]
df_Test$Activity_Desc <- ActLabels$V2[df_Test$Activity_id]
```

##### Merge the final data frame
```{r}
df_Final <- rbind(df_Train,df_Test)
```
##### Remove partials datasets
```{r}
rm(df_Train)
rm(df_Test)
```

##### Determine the variables of the data frame we need (mean and std)
```{r}
misColsMean <- grep("mean", colnames(df_Final), ignore.case=T, fixed=F, value = T)
misColsstd <- grep("std", colnames(df_Final), ignore.case=T, fixed=F, value = T)
```

##### Construct a vector with al variables needed
```{r}
misCols1 <- c("Subject_id", "Activity_id", "Activity_Desc")
misCols <- c(misCols1, misColsMean, misColsstd)
```

##### remove patial vectors
```{r}
rm(misColsMean)
rm(misColsstd)
```

##### Remove the variables of the data frame we don't need and keep all we need
```{r}
tdata <- df_Final[,(colnames(df_Final) %in% misCols)]
```

##### Transform the data from "tdata" to create the structure necesary to calculate de mean of each group of (subject, activity and variable "measure")
```{r}
z <- melt(tdata,id = misCols1, measure.vars = 3:88, variable.name = "measure", value.name = "value")
```

##### Calculate the mean of all grouped variables while arrange the structure
```{r}
tdatafin <- dcast(z, Subject_id + Activity_id + Activity_Desc ~ measure, mean)
```

##### Create tidyData.txt a txt file with the final tidy data
```{r}
write.table(tdatafin, file = "tidyData.txt", row.name = F, sep=",", append = F)
```
