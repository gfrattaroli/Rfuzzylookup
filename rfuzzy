##############################################################################
############ EDUBASE FUZZY LOOKUP contact:gfrattaroli@apple.com  18/10/2018 last updated
##############################################################################

#### FOR FIRST TIME USERS PLEASE READ THE 'README.rtf' file

#### All packages need to be installed
if("data.table" %in% rownames(installed.packages()) == FALSE) {install.packages("data.table")}
library(data.table)

if("tm" %in% rownames(installed.packages()) == FALSE) {install.packages("tm")}
library(tm)

if("RecordLinkage" %in% rownames(installed.packages()) == FALSE) {install.packages("RecordLinkage")}
library(RecordLinkage)

if("stringdist" %in% rownames(installed.packages()) == FALSE) {install.packages("stringdist")}
library(stringdist)

if("fuzzyjoin" %in% rownames(installed.packages()) == FALSE) {install.packages("fuzzyjoin")}
library(fuzzyjoin)

if("svMisc" %in% rownames(installed.packages()) == FALSE) {install.packages("svMisc")}
library(svMisc)

if("tidyverse" %in% rownames(installed.packages()) == FALSE) {install.packages("tidyverse")}
library(tidyverse)

if("lubridate" %in% rownames(installed.packages()) == FALSE) {install.packages("lubridate")}
library(lubridate)

if("openxlsx" %in% rownames(installed.packages()) == FALSE) {install.packages("openxlsx")}
library(openxlsx)

if("foreach" %in% rownames(installed.packages()) == FALSE) {install.packages("foreach")}
library(foreach)

if("RCurl" %in% rownames(installed.packages()) == FALSE) {install.packages("RCurl")}
library(RCurl)


###### Fetching data - This part gets data directly from the website, no need to change anything in here, unless the URL from edubase changes.
###### The CRT data needs to have this name 'CRT_CustomerData*_GB.xlsx' (no apostrophes)

Date <- Sys.Date()  ### Today's Date
datestring <- format(Date, '%Y%m%d') ### Format Today's Date as it fits the file name

EduBaseURL <- 'http://ea-edubase-api-prod.azurewebsites.net/edubase/edubasealldata'   ### Part of the URL - Base
EdusuffixURL <- '.csv' ### Part of the URL - Suffix
Edurl2fetch <- paste0(EduBaseURL, datestring, EdusuffixURL) ### Pasting together the base, the formatted datestring and the suffix

myfile <- getURL(Edurl2fetch, ssl.verifyhost=FALSE, ssl.verifypeer=FALSE) ### Store the connection into a variable
edubase <- read.csv(textConnection(myfile), header=TRUE, sep=",",stringsAsFactors = FALSE,encoding = 'latin1') ### Reading the CSV directly from the source

#### Doing the same for the links file
LinksBaseURL <- 'http://ea-edubase-api-prod.azurewebsites.net/edubase/links_edubasealldata'
LinksUrl2fetch <- paste0(LinksBaseURL, datestring, EdusuffixURL)

myfile <- getURL(LinksUrl2fetch, ssl.verifyhost=FALSE, ssl.verifypeer=FALSE)
edulink <- read.csv(textConnection(myfile), header=TRUE, sep=",",stringsAsFactors = FALSE,encoding = 'latin1')
####

CRT = read.xlsx(xlsxFile = 'CRT_CustomerData*_GB.xlsx', sheet = 1, skipEmptyRows = TRUE,detectDates = TRUE) ### Reading from CRT data

CRT$index.x <- seq.int(nrow(CRT)) ### Creating an index for CRT
edubase$index.y <- seq.int(nrow(edubase)) ### Creating an index for Edubase
edulink$index <- seq.int(nrow(edulink)) ### Creating an index for Edulink

####### Filtering edubase - Getting the postcode prefixes and removing the Scottish or NI ones. Also filtering by schools that are Open only.

edubaseTemp <- edubase[(c(5,11,60,63,65,134))] ### Extracting the column that we want from edubase into another dataframe
edubaseTemp$PostcodePrefix <- substr(edubaseTemp$Postcode,1,2) ### Getting the first two characters for each string
edubaseTemp$PostcodePrefix <- gsub('[0-9]+', '', edubaseTemp$PostcodePrefix) ### Removing any numerical character
ScottishNIPcodes <- c('AB','BT','DD','DG','EH','FK','G','HS','IV','KA','KW','KY','ML','PA','PH','TD','Z') ### Making a list of Scottish and NI pcodes
edubaseTemp <- edubaseTemp[!(edubaseTemp$PostcodePrefix %in% ScottishNIPcodes),] ### Get the schools that DON't have these Prefixes
edubaseTemp <- edubaseTemp[(edubaseTemp$EstablishmentStatus..name.=='Open'),] ### Get the schools that are open

####### Filtering CRT - Getting the postcode prefixes and filtering out all the Scottish or NI schools from matching.

CrTTemp <- CRT[(c(2,3,4,5,21))] ### Extracting the column that we want from Crt into another dataframe
CrTTemp$PostcodePrefix <- substr(CrTTemp$POSTCODE,1,2) ### Getting the first two characters for each string
CrTTemp$PostcodePrefix <- gsub('[0-9]+', '', CrTTemp$PostcodePrefix) ## Removing any numerical character
CrTTemp <- CrTTemp[!(CrTTemp$PostcodePrefix %in% ScottishNIPcodes),] ### Get the schools that DON't have these Prefixes

####### Preparing for Lookup - Collating the School Name, Address and Postcode into one column, also getting a list of Prefixes that are present in both databases.

EduPPfix <- unique(edubaseTemp$PostcodePrefix) ### All Unique Postcode Prefix from edubase (temporary database)
CrTPPfix <- unique(CrTTemp$PostcodePrefix) ### All Unique Postcode Prefix from CRT (temporary database)
PPfix <- CrTPPfix[(CrTPPfix %in% EduPPfix)] ### All Postcodes that are in CRT and in edubase
PPfix <- PPfix[!(is.na(PPfix))] ### Removing NA in the list

CrTTemp$paste <- paste(CrTTemp$COMPANY,CrTTemp$ADDRESS1,CrTTemp$POSTCODE) ### Making of the paste column, this is the column which the lookup is going to be
CrTTemp$paste <- tolower(CrTTemp$paste) ### making all lower case, improving matching

edubaseTemp$paste <- paste(edubaseTemp$EstablishmentName,edubaseTemp$Street,edubaseTemp$Postcode) ### Making of the paste column, this is the column which the lookup is going to be
edubaseTemp$paste <- tolower(edubaseTemp$paste) ### making all lower case, improving matching

####### 

CRT_V <- CrTTemp[(c(5,6,7))]  #### Getting just the index, Postcodeprefix and the paste column
edu_V <- edubaseTemp[(c(6,7,8))]

Interval <- 55  ### Minimum confidence Interval, change this number at your will, the higher the less chance of an error.

######
RealInterval <- 1-(Interval/100) ### Change the interval into a number acceptable for the algorithm
Finaldb2 <- 0 ### Set Finaldb2 as an empty value
DbName <- 0 ### Set DbName as an empty value
foreach(x=1: length(PPfix)) %dopar% {   #### This for loop is to use the algorithm in parallel reducing its processing time considerably
  CRT_V1 <- CRT_V %>%
    filter(CRT_V$PostcodePrefix==PPfix[x])  ### Filter Crt on the x postcode prefix value
  edu_V1 <- edu_V %>%
    filter(edu_V$PostcodePrefix==PPfix[x])  ### Filter edubase on the x postcode prefix value
  DbName <- stringdist_left_join(CRT_V1, edu_V1, by='paste', method='jaccard', max_dist=RealInterval, distance_col ="dist",q=3)  ### This matches the two paste column using the Jaccard index
  if("dist" %in% colnames(DbName)) ### If there is a dist column in the matched db (Might happen that there were no matches for that postcode prefix)
  {
    DbName$dist <- 1-DbName$dist ### Change the distance number into the confidence interval percentage
    DbName <- DbName[order(DbName$index.x,-DbName$dist),] # Order on Crt index and on confidence interval descending order
    DbName <- DbName[!duplicated(DbName$index.x),] # Remove all CRT duplicates
    DbName <- DbName[!duplicated(DbName$index.y),] # Remove all Edubase duplicates
    Finaldb2 <- rbind(Finaldb2,DbName) # Append every postcode results to the main database
  } else { #if there is not a distance column
    DbName$dist <- 0 # set distance as 0
    DbName <- DbName[order(DbName$index.x,-DbName$dist),] #order by Crt index and dist
    Finaldb2 <- rbind(Finaldb2,DbName) # Append the db to the main one
  }
}


Finaldb2 <- Finaldb2[(c(1,4,7))] #### Getting the distance column and the two indexes

MatchedDB <- merge(CRT,Finaldb2,by='index.x', all.x=TRUE)  #### Merging by index with the original datasets
MatchedDB <- merge(MatchedDB,edubase,by='index.y', all.x=TRUE)

########## Merging with the URNs - All non-matched rows will be merged by URN (only if they have 'U' as first character of their professional number)
########## with the rest of edubase

NonMatchedDB <- MatchedDB[is.na(MatchedDB$index.y),] ### Getting all the non-matched result

MatchedDB <- MatchedDB[!is.na(MatchedDB$index.y),] ### Getting all the matched results

NonMatchedDB <- NonMatchedDB[,colSums(is.na(NonMatchedDB))<nrow(NonMatchedDB)] ### Removing all empty columns

x = substring(NonMatchedDB$PROF_NUMBER, 1, 1) ### Setting x as a check of the first character in Prof.Number column

NonMatchedDB$URN <- ifelse(x=='U',gsub("[^0-9\\.]", "", NonMatchedDB$PROF_NUMBER),'') ### If the first character is a U then remove all non-numeric characters

NonMatchedDB <- merge(NonMatchedDB,edubase,by='URN', all.x=TRUE) ### Merge the non matched entries with edubase

FinalDB <- rbind(MatchedDB,NonMatchedDB) ### Build the final file adding Matched and non matched entries

##### Merging with link and writing the final file

FinalDB <- merge(FinalDB,edulink,by='URN',all.x=TRUE)

FinalDB <- FinalDB[order(FinalDB$index.x),]

wb <- createWorkbook()
addWorksheet(wb = wb, sheetName = 'FinalDB')
writeData(wb, sheet=1, FinalDB)
saveWorkbook(wb, 'FinalDB.xlsx', overwrite = TRUE)

###### Output the unmatched EduBase entities 

Unmatched_Edubase <- edubase[!(edubase$index.y %in% FinalDB$index.y),]

wb <- createWorkbook()
addWorksheet(wb = wb, sheetName = 'Unmatched_Edubase')
writeData(wb, sheet=1, Unmatched_Edubase)
saveWorkbook(wb, 'Unmatched_Edubase.xlsx', overwrite = TRUE)







