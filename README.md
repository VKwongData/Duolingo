# Duolingo Data ETL Using Alteryx Designer

This is a repository for my portion of the final project for the course CIS9440 Data Warehousing and Analytics (Spring 2022 semester). As part of Group 5, I was solely responsible for the ETL steps in the project, which involved using Alteryx Designer to clean and transform the data so that the data can be loaded into a Microsoft Azure SQL Database. The following are descriptions of the files within this repository:

<ul>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/Data%20Warehousing%20Group%20Project%20-%20Tables%20.yxmd">Data Warehousing Group Project - Tables.yxmd</a>: The Alteryx workflow created for the project.</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/learning_traces.13m.csv">learning_traces.13m.csv</a>: The input file that has the Duolingo data. The project originally used a dataset that had approximately 12.8 million rows. Due to the original file's enormous size, the file that is in this repository contains only the first 200,000 rows of data.</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/lexeme_reference.txt">lexeme_reference</a>: The input file that has definitions for abbreviations used in the learning_traces.13m.csv file. This file was used in the Alteryx workflow to transform the data into something more understandable.
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/CLEAN%20SAMPLE%20learning_traces.csv">CLEAN SAMPLE learning_traces.csv</a>: An output file from the Alteryx workflow that shows the transformed data in a csv file before it was loaded into the Microsoft Azure SQL Database.</li>
<li><a href="https://github.com/VKwongData/Duolingo/blob/main/Presentation%20-%20Selected%20Slides.pptx">Presentation - Selected Slides</a>: Slides from the group presentation that introduced the data and outlined the ETL process. All slides were created by me.</li>
</ul>

## Introduction

Duolingo is a mobile application and desktop website that allows any person to access all its language-learning content for free, and while Duolingo also offers a paid, ad-free version that includes multiple benefits that allow a user to progress more quickly in a language, most users access its content via the free, ad-supported version. Duolingo currently offers 40 languages to learn and has 24 user interface languages. The breadth of both access and content has allowed Duolingo to dominate the language-learning mobile application global market as the most downloaded education app in the world.

The dataset being processed by the Alteryx workflow is data gathered by Duolingo originally to assist with an experiment on Half-Life Regression (“HLR”). The dataset originally consisted of 12,854,226 learning traces taken over two weeks in 2013 from a group of anonymized users (the dataset in this repository consists of the first 200,000 learning traces in the dataset). Duolingo contributed to the dataset to support a paper on Half-Life Regression, a model for spaced repetition, and the research team subsequently released the dataset to the public to facilitate future research into language learning. The dataset of learning traces was released on <a href="https://dataverse.harvard.edu/dataset.xhtml?persistentId=doi:10.7910/DVN/N8XJME">Harvard Dataverse</a>, while the metadata, lexeme tag references, and experiment code were documented on <a href="https://github.com/duolingo/halflife-regression">GitHub</a>. 

## Methodology

I used Alteryx Designer to clean the dataset so that the data could be readily loaded into a Microsoft Azure SQL Database. I then conducted further cleaning of the data by replacing any abbreviations or tags with the full words or definitions, using the “lexeme_reference.txt” file from GitHub. Six dimensional tables and two fact tables were created in Alteryx Designer and loaded into a Microsoft Azure SQL Database. 

Below are the initial steps of our ETL process:

 ![Picture1](https://user-images.githubusercontent.com/94913441/181410951-de7774aa-3407-4bce-85eb-e610fc657ffc.png)

After bringing the learning_traces.13m.csv file into the workflow, I focused on the lexeme_string attribute in the dataset. This attribute had information about the word in the form of tags: the surface form, lemma, part of speech and morphological components. I separated the lexeme (i.e., surface form/inflected form), lemma and part-of-speech tags, and created an attribute to store all the morphological components. Then the Alteryx workflow counted the number of characters of the surface form and the number of morphological components and put the results into two new attributes. For the project, our group used the number of characters and number of morphological component tags as proxies for a given word’s difficulty. For example, if a person were learning the English word “and”, Alteryx Designer would calculate that the word has three characters and no morphological components. However, a shorter word such as the English word “is”, which has two characters, might have been considered more difficult in our analysis because we took into consideration that the word had three morphological component tags: “&lt;pri&gt;&lt;p3&gt;&lt;sg&gt;”, which denote present indicative, third person and singular, respectively. 

Below is a sample of the result of this portion of the ETL process:
 
![Picture2](https://user-images.githubusercontent.com/94913441/181411016-ac65484b-54ad-46af-b709-c209f33e1af1.png)

I then brought the lexeme_reference text file into a Find Replace Tool.

![Picture3](https://user-images.githubusercontent.com/94913441/181411097-d51fabcd-f08f-49c1-9e0f-49e558f658c9.png)

After bringing in the lexeme_reference text file, I replaced the part-of-speech tags with something more understandable. I then created an attribute for the number of characters of the surface form (i.e., “lexeme_length”), which was another one of our proxies to measure the difficulty of a word. Below is a sample of the result of this portion of the ETL process:
 
![Picture4](https://user-images.githubusercontent.com/94913441/181411253-0f6da3bc-b634-4ec3-b83d-73c0c78bfa2f.png)

I also replaced all language abbreviations with the full names of the languages. Below is a sample of the result of this portion of the ETL process:

![Picture5](https://user-images.githubusercontent.com/94913441/181411731-3d686532-3fea-4d58-92ff-710326494630.png)
 
I noticed that the User IDs in the dataset were case sensitive (e.g., “u:FO” and “U:fo” would be considered different users). To take care of case insensitivity issues later in the project, I created a user_id attribute to use going forward as a unique id for each user.

 ![Picture6](https://user-images.githubusercontent.com/94913441/181411972-0fa572c8-5911-4bbb-bb52-4802738bc80d.png)

Below is a sample of the result of this portion of the ETL process:
 
![Picture7](https://user-images.githubusercontent.com/94913441/181412029-670ed01b-dc60-46a1-9c86-48ecb17d1d1c.png)

At this point in the ETL process, the workflow was ready to create the dimension and fact tables that will be loaded into the Microsoft Azure SQL Database. I used Block Until Done Tools to organize the order in which the tables were created. Below is the part of the workflow that created the pos_dim, lexeme_dim and lemma_dim tables:

![Picture8](https://user-images.githubusercontent.com/94913441/181412191-9f62380e-1f29-4b0d-96bd-6052f6007331.png)
 
As seen above, for any dimensions that did not already have a unique ID attribute that I could use in the dataset, I created an ID attribute to serve as the Primary Key for its dimension table. I also used the “Post Create SQL Statement” option in the Output Data Tool to designate the Primary Key for each dimension and then loaded each table to the database. A sample of the SQL code is below:

![Picture9](https://user-images.githubusercontent.com/94913441/181412634-0c1519bd-0f8c-4eec-ad2c-8b4792079599.png)
 
Below is the part of the workflow that created the language_dim, user_dim and timestamp_dim tables: 

![Picture10](https://user-images.githubusercontent.com/94913441/181413010-9065237c-f825-434e-b22d-f2c9efed71c7.png)

As seen above, the language and timestamp dimensions had an extra step involved before applying the same steps as for creating the other dimension tables:

<ul>
<li>Language Dimension: I had to use the Union Tool to include both the user interface languages and the learning languages in the same dimension.</li>
<li>Timestamp Dimension: I had to use the Formula Tool to convert the timestamps from UNIX to a DateTime data type.</li>
</ul>

Below is the part of the workflow that created the two fact tables for the project:

![Picture11](https://user-images.githubusercontent.com/94913441/181413557-b6e4d140-8132-4b36-8d37-290165a3bdd1.png)
 
Finally, I used the “Post Create SQL Statement” option in the Output Data Tool to designate the Foreign Keys for each fact table. A sample of the SQL code is below:

 ![Picture12](https://user-images.githubusercontent.com/94913441/181413850-508a9829-0f2e-47a1-8dea-05185b94c3c7.png)

The above Alteryx workflow is repeatable. The extraction and transformation time takes about 3 minutes using the full dataset of around 13 million rows on a personal laptop, while the loading time into the database is based on internet connection and Microsoft Azure SQL Database service tier. 
