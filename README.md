# wiki-solution-package

This Solution uses Spring Boot , Apache Kafka and Cassandra , this package only contain docker compose file and it uses published image of necessary software.

Code corresponding to solution exists at following repository:
https://github.com/pratik1788/wiki-coordinator
https://github.com/pratik1788/wiki-reader 
https://github.com/pratik1788/wiki-sink


Architecture Diagram: https://www.edrawmax.com/online/share.html?code=9ea080fa0a0c11ec84e80a54be41f961  ( Please refresh if it doean't load first time)

## Architecture Summary of Component

### Wiki-Coordinator :
This is Interaction layer for over all solution.  Request to Load file is made via this app by publishing message to Kafka . THis app also tracks status of process by reading notification messages and perisiting it to cassandra db.

### Wiki-Reader :
This component reads kafka notification to start reading file and publish rows in batch ( configurable ) to  kafka . ( THis whole process works without actually downloading file into local/temp)

### Wiki-sink :
Reads message published by reader and persists to Cassandra.


### Note:  

To make oversolution work, cassandra is used from cloud provider , configuration is 1 Node , with 8 GB RAM and 160 GB SSD , Ideally solution could be achieved without that but I ran into resource challange on my workstation and hence decided to use cloud cassandra.

### Improvement Scope:

1) Addition of Dead Message Queue / Table ( I am yet to add correct processing of failed message due to reasons like invalid row from file to cassandra having issue )
2) Choice of data model for cassandra , currently data is partitioned by year month day hour and language to make queries optimal that uses language. I started with though process of having single partition for all data , but i have seen query performance issue . I need to look more into.
3) Containerize cassandra as well with solution.


## Deployment/run steps

### Pre Requisite
This Solution requires docker software available on system for local run. Steps to download docker Docker Installation Step

Docker Image for required component open source ( Kafka , zookeeper) developed code ( wiki-coordinator, wiki-reader, wiki-sink ) already pushed to docker repo.

### Deployment Step
#### Step 1 :

Download docker compose file that combines required component to overall solution to run. docker compose file is available within wiki-solution-package . Link for same is here : https://github.com/pratik1788/wiki-solution-package/blob/main/docker-compose.yml & ensure that docker engine is running.

#### Step 2: 
Using Terminal/Command Prompt Navigate to folder where above docker-compose file has been downloaded.

#### Step 3: 
Run "docker-compose up -d"

#### Step 4: 
Validate that docker-coordinator app is fully up and connected with kafka ( as docker initialize image together , there will be few seconds till kafka is available and till that time app will retry to connect kafka. ( It is observed taht some time kafka/zookeeper is shutting down abruptly , in that case please run "docker-compose stop" and try to up it again using above command)

#### Step 5 : 
Open browser nad enter url : http://localhost:8080/swagger-ui.html#/

####  If swagger page is loaded then app is running.

## Steps to import file

#### Step 1: 
go to http://localhost:8080/swagger-ui.html#/

#### Step 2: 
click on /import-request Post link. & enter payload as below to load file for 2012 year 01 month 01 day and 0th hour data

{ "filenameToExtract": "pagecounts-20120101-000000.gz" }

Expected Response body should show message as "Request to Load file has been successfully made. Please check import-status to get latest update"

#### Step 3: 
Track Import Status using /import-status/{fileName}. & enter same file name as parameter "pagecounts-20120101-000000.gz"

Response will lay down steps that are completed, look for "dataSinkSuccessful" as event name to ensure that file load is complete. It is observed that file load of 5 M record is taking around 20 minute.

## Steps to Analyze Data

#### Step 1 : find Top 10 Page by Language go to data-analyzer-controller & click on /getTop10byLanguage/{yearMonthDay}/{hour}}

enter yearMonthDay as 20120101
hour as 0

Response will contain desired records in json format.


## Steps to Build solution and push image to docker ( Not Needed unless we want to make code change and re build )

#### Overall Solution consists of 3 Repository listed as below

https://github.com/pratik1788/wiki-coordinator
https://github.com/pratik1788/wiki-reader 
https://github.com/pratik1788/wiki-sink

#### Pre Requisite : 

Check out all repository , to build and run test , prerequisite is to have maven installed , to check if maven is installed or not please use command "mvn -v" and validate that it runs successfully and provide version .


#### Step 1: 

navigate to folder where wiki-coordinator is downloaded and run "mvn clean package " command to build , run test and package application. 

#### Step 2: 
Run command Docker build -t wiki-coordinator . 

#### Step 3: 
( Optional in case we want to publish image to repository ) : Run "docker images" command & get image id of wiki-coordinator Run "docker image tag fab3b82c7e66 pratik1788/wiki-coordinator" Run "docker image push pratik1788/wiki-coordinator" command

#### Step 4 & 5 : Repeat same steps for wiki-reader and wiki-sink

