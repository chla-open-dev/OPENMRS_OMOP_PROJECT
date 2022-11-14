# OPENMRS_OMOP_PROJECT
Welcome to our OpenMRS-OMOP project! 

We are a team of physicians, developers, and researchers at Children's Hospital Los Angeles (CHLA) named OPEN. For more information about our work, please visit www.learnwithopen.org & www.avetis.org. If you have any questions, our email is info@learnwithopen.org.

*Project Summary* 

As more low and middle-income countries (LMICs) implement electronic health record systems (EHRs), informatics has become an important component of global health. OpenMRS is a popular open-source EHR that has been implemented in over 60 countries. As in high income countries, interoperability and research capabilities remain a challenge. The Observational Medical Outcomes Partnership (OMOP) is one of the most relevant common data models (CDM) to support EHR-based research and data sharing, but its adoption has been limited in LMICs. To address this gap, we developed an OpenMRS to OMOP extract, transform, and load (ETL) tool using Talend. We built on existing documentation to develop a comprehensive concept map from OpenMRS to OMOP. The OMOP domains were reviewed for overlapping concepts in OpenMRS, and a core set of tables were selected for ETL development. Specific variables were then identified from OpenMRS tables which mapped to OMOP domain fields. Afterwards, the ETL tool was developed using MySQL Workbench, PostgreSQL, and Talend.

*Tutorial* 

For a complete tutorial with screenshots, please download this [word document](https://github.com/chla-open-dev/OPENMRS_OMOP_PROJECT/blob/main/Documentation/SetupOmopDocument.docx) from our GitHub files. You can also view a limited version of the [below](https://github.com/chla-open-dev/OPENMRS_OMOP_PROJECT/wiki/Tutorial-Guide:-OpenMRS-to-OMOP-CDM).

# ETL Tutorial - Moving Data from OpenMRS to OMOP CDM using Talend
*by Dr. Barry Levine*


Overview:
1.	Create the openmrs database	3
2.	Set up pgAdmin	3
3.	Create the PostgreSQL database	3
4.	Download the ATHENA SNOMED VOCABULARY	5
5.	Set up the Common-Data-Model/PostgreSQL database and populate the database with the Snomed Vocabulary	6
6.	Set up Talend – create jobs to move data from source to target	20

The ETL scripts we used were based on the following document:
https://wiki.openmrs.org/download/attachments/84476300/OpenMRS%20ETL.docx?version=1&modificationDate=1436743360000&api=v2 
The document was referenced by Paul Biondich in his post at:
https://talk.openmrs.org/t/openmrs-and-ohdsi/24930

The source is a mysql sample openmrs database included in the install of OpenMRS version 2.1.4. The target is a PostgreSQL database – we used PostgreSQL 13.

In this document we will describe the ETL process using Talend Open Studio for Data Integration version 7.3.1.
(https://www.talend.com/home/). Note, we used Mac computers for this process, although it should work well on a PC.

The final piece of software to set up is pgAdmin 4; other more recent versions of pgAdmin should work well.

We are using Java JDK 8

An overview of the steps to set up the ETL system are
1.	Create the openmrs database
2.	Set up pgAdmin
3.	Create the PostgreSQL database
4.	Download the ATHENA SNOMED VOCABULARY
5.	Set up the Common-Data-Model/PostgreSQL database and populate the database with the Snomed Vocabulary
6.	Set up Talend – create jobs to move data from source to target

The remainder of this document will consist of elaborations of each of the 6 steps, above.

1.	Create the openmrs database
Set up OpenMRS with a demo database. 

2.	Set up pgAdmin
pgAdmin download can be found here: https://www.pgadmin.org/

3.	Create the PostgreSQL database
Choose an appropriate name for the target database. In the example provided herein we used op1

 
4.	Download the ATHENA SNOMED VOCABULARY

Download ATHENA SNOMED VOCABULARY from
https://athena.ohdsi.org/vocabulary/list
 

Be sure to check SNOMED, then Download Vocabularies
Then, wait until the download is ready at https://athena.ohdsi.org/vocabulary/download-history:

 

The download will create a zip file like vocabulary_download_v5_{8c5131e2-9350-4549-a813-3da924313182}_1613867923031.zip

Unzip this file. You’ll see a file structure similar to the following:
 
 
5.	Set up the Common-Data-Model/PostgreSQL database and populate the database with the Snomed Vocabulary

Refer to the following steps 
Browse to https://github.com/chla-open-dev/OPENMRS_OMOP_PROJECT/tree/main/PostgreSQL

 
Follow the steps listed above
1.	Already created schema (op1)
2.	Select script file and execute in pgadmin:
 
Open query tool, below

 
 

After the script executes you can see the tables that have been created:

 


3.	Load your data into the schema

 
Select VocabImport to get the sql query for loading the data

 
This file contains the following queries:

 

You will need to modify the file paths in this .sql file to reflect where you downloaded the Athena Snomed Vocabulary.

For example, I modified the file as follows:

COPY DRUG_STRENGTH FROM '/Users/xxx/OpenMRS/xxx/OMOP/xxx/PostgresSetup/AthenaVocabularies/CDMV5VOCAB/DRUG_STRENGTH.csv' WITH DELIMITER E'\t' CSV HEADER QUOTE E'\b' ;

COPY CONCEPT FROM '/Users/xxx/OpenMRS/xxx/OMOP/xxx/PostgresSetup/AthenaVocabularies/CDMV5VOCAB/CONCEPT.csv' WITH DELIMITER E'\t' CSV HEADER QUOTE E'\b' ;

…

Be sure to execute each query, above, in a query window.
It will take about 10 minutes to complete.



 
4.	Execute the script OMOP CDM postgresql pk indexes.txt to add the minimum set of indexes and primary keys we recommend.
5.	Execute the script OMOP CDM postgresql constraints.txt to add the constraints (foreign keys).
6.	Ignore this step

Next, we need to modify some of the PostgreSQL tables in the op2 schema.

Modify location table to match the following:
 

 



observation_period
 

 

 
 

 

CREATE TABLE “death” to record patient deaths:

 

 
 
6.	Set up Talend – create jobs to move data from source to target

In this section we will provide an introduction to setting up Talend to move the OpenMRS demo database (source, mysql) to the OMOP database (target, PostgreSQL).

Start Talend and you will see something like:

 

You will import our project. Then browse to the folder containing the project. The project is found here: https://github.com/chla-open-dev/OPENMRS_OMOP_PROJECT
The name of the project is CHLA_OMOP_PROJECT. Download the zip file and unzip it in your directory of choice (this will be the project root directory – see below).


 

Select the Code button. Download the entire folder contents as a zip file:

 


 

We’re interested in the folder CHLA_OMOP_PROJECT, but without the readme.
 

Next, select “Finish”.


 


 

Select CHLA_OMOP_PROJECT – java, then select “Finish”


 

Select “Start now!”

 

 


The ETL jobs must be run in the following order:
1.	PostgreSQL_Location 0.1
2.	PostgreSQL_Care_Site 0.1
3.	PostgreSQL_Person 0.1
4.	PostgreSQL_Provider 0.1
5.	PostgreSQL_Observation_Period 0.1
6.	PostgreSQL_Observation 0.1
7.	PostgreSQL_Person_death 0.1

Note: the source is a Mysql 5 database and the target is a PostgreSQL database, as indicated in the previous section. 

Source Connection Information:
Database: MySQL
DB Version: Mysql 5
Host: 127.0.0.1
Port: 3306
Database Name: openmrs

Target Connection Information:
Database: PostgreSQL
DB Version: v9 and later
Host: localhost
Port: 5432
Database Name: op1


The source/target connection information as specified in the jobs must be modified appropriately to reflect the properties you are using.

For instance, consider the connection information as specified in the PostgreSQL_Care_Site 0.1 job.

 

Double click on PostgreSQL_Care_Site 0.1, above, to display the job details:

 

Click on the location icon to display its properties in the Component tab panel below:

 


Edit the connection information (Host, Port, etc.) to reflect the properties you are using. Also, note the query used to collect the source database information to transfer to the PostgreSQL database.
Repeat this process for all of the jobs. It is strongly suggested you frequently save your edited properties using the File/Save Talend menu item.

The same process should be used for all target database icons, as well. For example, in the job PostgreSQL_Care_Site 0.1 you will need to edit the connection properties for both the location_id_lookup and care_site icons.

 

 

 
In addition to updating the connections in the jobs it’s also necessary to update the metadata connection properties. We use the metadata items in the job PostgreSQL_Observation 0.1

 

 

Double click on metadata item openmrs_db 0.1 to yield

 

Select Next

 

Edit the connection properties as above. Then, select Finish.
Note, the database name appears both in the connection string and DataBase name fields.

 

Select Yes to propagate modification.

Edit the properties for postgresql_db 0.1, as well.

 

Next, fix the connections in the job:

 

Note, the adjustment to the schema name above.
 
After updating all of the connection properties the jobs can be run. Recall, the first job to run is PostgreSQL_Location 0.1

Jobs are run by:
1.	Select the job by double clicking on it in the left-hand panel

 


 

Click on the Run button. Upon completion of the run you will see the following:
 

Note the Exit code = 0 in the Run panel and the rows moved in the upper panel.



















Notes:

 


The query in the source obs table (move job PostgreSQL_Observation 0.1) sets a limit at 1000 obs. 

Once everything looks good then we want to remove the limit on the number of observations being moved over to PostgreSQL.

Finally, if an OpenMRS concept does not have an exact match in Snomed then we store “0” in the obs table for obs_concept_id.

 

The intent of this tutorial is to demonstrate how to set up and run the Talend ETL tool to run the jobs we created. We suggest you check the available Talend tutorials to expand on our work, e.g., when creating jobs corresponding to other OpenMRS tables. 

