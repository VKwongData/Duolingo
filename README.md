# Duolingo Data ETL Using Alteryx Designer

This is a repository for my portion of the final project for the course CIS9440 Data Warehousing and Analytics (Spring 2022 semester) at the Zicklin School of Business. As part of Group 5, I was solely responsible for the ETL portion of the project, which involved using Alteryx Designer to clean and transform the data so that the data can be loaded into a Microsoft Azure SQL Database.

## Introduction

The dataset being processed by the Alteryx workflow is data gathered by Duolingo originally to assist with an experiment on Half-Life Regression. The dataset originally consisted of 12,854,226 learning traces taken over two weeks in 2013 from a group of anonymized users (the dataset in this repository consists of the first 200,000 learning traces in the dataset). The research team subsequently released the dataset to the public to facilitate future research into language learning. The dataset of learning traces was released on <a href="https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME">Harvard Dataverse</a>, while the metadata, lexeme tag references and experiment code were documented on <a href="https://github.com/duolingo/halflife-regression">GitHub</a>. 

## Methodology

I used Alteryx Designer to transform the dataset so that the data could be readily loaded into a Microsoft Azure SQL Database as six dimension tables and two fact tables, as per the requirements of the class project. 

Below are the initial steps of our workflow:

 ![Picture1](https://user-images.githubusercontent.com/94913441/181410951-de7774aa-3407-4bce-85eb-e610fc657ffc.png)

After bringing the learning_traces.13m.csv file into the workflow, I focused on the lexeme_string attribute in the dataset. This attribute had information in the form of tags about the word's surface form (i.e., lexeme), lemma, part of speech and morphological components. In the workflow, the lexeme, lemma and part-of-speech tags were separated into their own attributes. Then, the workflow counted the number of morphological components and put the results into a new attribute.

Below is a sample of the result of this portion of the workflow:
 
![Picture2](https://user-images.githubusercontent.com/94913441/181411016-ac65484b-54ad-46af-b709-c209f33e1af1.png)

Continuing onwards in the workflow, the lexeme_reference.txt file was brought into a Find Replace Tool:

![Picture3](https://user-images.githubusercontent.com/94913441/181411097-d51fabcd-f08f-49c1-9e0f-49e558f658c9.png)

The Find Replace Tool replaced the values in the part-of-speech attribute with something more understandable, and an attribute (called lexeme_length) was created for the number of characters of the surface form. Below is a sample of the result of this portion of the workflow:
 
![Picture4](https://user-images.githubusercontent.com/94913441/181411253-0f6da3bc-b634-4ec3-b83d-73c0c78bfa2f.png)

All language abbreviations in the two language attributes were also replaced with the full names of the languages. Below is a sample of the result of this portion of the workflow:

![Picture5](https://user-images.githubusercontent.com/94913441/181411731-3d686532-3fea-4d58-92ff-710326494630.png)

At this point in the workflow, most of the transformations of the lexeme_string and language columns were done. 
 
I noticed that the User IDs in the dataset were case sensitive (e.g., “u:FO” and “U:fo” would be considered different users). To take care of case insensitivity issues later in the project, a user_id attribute (numbering users from 1 onwards) was created with a unique id for each user.

 ![Picture6](https://user-images.githubusercontent.com/94913441/181411972-0fa572c8-5911-4bbb-bb52-4802738bc80d.png)

Below is a sample of the result of this portion of the workflow:
 
![Picture7](https://user-images.githubusercontent.com/94913441/181412029-670ed01b-dc60-46a1-9c86-48ecb17d1d1c.png)

Finally, the workflow was ready to create the dimension and fact tables that will be loaded into the Microsoft Azure SQL Database. Block Until Done Tools were used to organize the order in which the tables were created. Below is the part of the workflow that created the pos_dim, lexeme_dim and lemma_dim tables:

![Picture8](https://user-images.githubusercontent.com/94913441/181412191-9f62380e-1f29-4b0d-96bd-6052f6007331.png)
 
As seen above, for any dimensions that did not already have a unique ID attribute that could be used in the dataset, an ID attribute was created to serve as the Primary Key for its dimension table. The “Post Create SQL Statement” option was used in each Output Data Tool to designate the Primary Key for each dimension, and then each Output Data Tool loaded a table into the database. A sample of the SQL code is below:

![Picture9](https://user-images.githubusercontent.com/94913441/181412634-0c1519bd-0f8c-4eec-ad2c-8b4792079599.png)
 
Continuing onwards in the workflow, below is the part of the workflow that created the language_dim, user_dim and timestamp_dim tables: 

![Picture10](https://user-images.githubusercontent.com/94913441/181413010-9065237c-f825-434e-b22d-f2c9efed71c7.png)

As seen above, the language and timestamp dimensions had an extra step involved before applying the same steps as for creating the other dimension tables:

<ul>
<li><b>Language Dimension:</b> The Union Tool was used so that languages from both language attributes were included in the same dimension.</li>
<li><b>Timestamp Dimension:</b> The Formula Tool was used to convert the timestamps from a UNIX timestamp to a DateTime data type.</li>
</ul>

Lastly, below is the part of the workflow that created the two fact tables for the project:

![Picture11](https://user-images.githubusercontent.com/94913441/181413557-b6e4d140-8132-4b36-8d37-290165a3bdd1.png)
 
The “Post Create SQL Statement” option in each Output Data Tool was used to designate the Foreign Keys for each fact table. A sample of the SQL code is below:

 ![Picture12](https://user-images.githubusercontent.com/94913441/181413850-508a9829-0f2e-47a1-8dea-05185b94c3c7.png)

All the steps outlined above are a repeatable Alteryx workflow that can be run in its entirety in a single run. Utilizing a personal laptop, the extraction and transformation time takes about 3 minutes when using the full dataset of around 12.8 million rows. The loading time into the database is based on internet connection and Microsoft Azure SQL Database service tier. 

### Files Included in Repository:

<ul>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/Data%20Warehousing%20Group%20Project%20-%20Tables.yxmd">Data Warehousing Group Project - Tables.yxmd</a>: The Alteryx workflow created for the project.</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/learning_traces.13m.csv">learning_traces.13m.csv</a>: The input file that has the Duolingo data. The project originally used a dataset that had approximately 12.8 million rows. Due to the original file's enormous size, the file that is in this repository contains only the first 200,000 rows of data.</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/lexeme_reference.txt">lexeme_reference.txt</a>: The input file that has the definitions for the abbreviations used in the learning_traces.13m.csv file.
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/CLEAN%20SAMPLE%20learning_traces.csv">CLEAN SAMPLE learning_traces.csv</a>: An output file from the Alteryx workflow that shows the transformed data in a single csv file (excluding transformations done to create the dimension and fact tables for the database).</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/Presentation%20-%20Selected%20Slides.pptx">Presentation - Selected Slides.pptx</a>: Selected slides from the group presentation that introduced the data and outlined the ETL process. All slides were created by me.</li>
</ul>
