# Transforming-and-Analyzing-Data-with-SQL
## Table of Contents

1. [Project/Goals](#projectgoals)
2. [Process](#process)
3. [Results](#results)
4. [Challenges](#challenges)
5. [Future Goals](#future-goals)

## Project/Goals
### Goals
The goal of this project was to practice implementing what we have learned in the course, including:
* Extracting data from a SQL database
* Cleaning, transforming and analyzing data
* Loading data into a database
* Developing and implementing a QA process to validate transformed data against raw data


## Process
### Step 1 - Data exploration
In order to start the process of transformaing and loading data into a usable format it is important to be familiarized with the data. 
A large portion of time was dedicated to going through each column of each table once they were imported into a database from their CSV files. Once in a SQL environment, it became easy to quickly view the tables with different filters, look at column summary values, and identify potential issues in the data. 
### Step 2 - Cleaning the data
The next step involved cleaning the date. This step can be broken down into further steps as well. 
1. Identify the issues with the data and decide on how to handle them
2. Complete the above step for each column and table as necessary
4. Create new tables and import the old, uncleaned tables into them. 
5. Apply all updates to new table so that the data becomes cleaned. 
### Step 3 - Using the cleaned data
Now that the data has been cleaned, we can use it to answer a variety of questions. This can be similar to the first step in the process, but with the goal of providing insights rather than identifying problems. 
### Step 4 - Identifying next steps
The dataset is still far from perfect. There may by a wide variety of issues with the current data. This means that further cleaning can be done to the database. 
Improving the database structure would also be an ideal next step. As it currently stands, the database does not have primary keys assigned, and therefore is not a true relational database. This leaves lots of room for error whenever combinging tables in order to provide insights. 

## Results
Without any previous knowledge of website traffic data, or google analytics, it was a good introduction to the type of data one may see in real life. It also made apparent the benefits of having a well maintained relational database, the importance of database architecture, and the importance of data collection. It was unclear how many sources the data came from but the discrepancies between tables implied it could have been multiple sources. Although this is not inherently an issues, it was clear that some thought must be put into the data collection to ensure consistent and productive results. 

## Challenges 
The largest challenge was mostly due to not being able to ask anyone about the meaning of the data. In the real world if you were unsure of a column name, or how a column was calculated, you could likely ask either the client, your superior or colleague. Without this knowledge many assumptions needed to be made. 

As a first project in the world of data science it was necessary to be building the bike as it was ridden, in the sense that an action plan needed to be devised at the same time as it was being executed. Reviewing my notes from the project, it is clear where my approach to a given problem, or a certain strategy changed over time. 

## Future Goals
There are essentially 3 main goals I would have liked to accomplished with more time. They are the follow:
1. Continue cleaning the date
	- this means every missing space, extra space, typo, unpadded id number, incorrect date and time format, and duplicate that still remains in the database
2. Next would be establishing a source of truth
	- there are many duplicates across tables, some of which agree with each other and others that do no
	- it's critical to know what data was most recently added, what values are out of date or no longer in use
	- from this, a primary key for a table could be created. From there, other primary keys could be identified or created
3. Turn it into a relational database
	- once the primary keys are established, it would be easier to create new tables, and reduce redundancy within the database
	- For example, there are some product names that can be broken down further into sub categories and colours. This is grounds for a much more details "products" table, with a unique identifier (primary key)
	- I envision 3-5 tables each with primary keys in order to complete this goal
