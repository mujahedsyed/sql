# sql questions

 - **ACID** Stands for atomicity, consistency, isolation, durability.
 - A Table is in **First Normal form** if it has no repeating fields or no repeating columns for example if you have table that stores users and their primary and secondary phone numbers than because you are storing more than one phone number fields this violates 1NF as you have repeating fields.
 ![Alt text](images/1nf.png?raw=true "1NF")
 - A table is in in **Second Normal Form** if it is in 1NF and has no attribute associated with partial key column or no attribute associated with part of primary key. For example in below table first three columns are primary key in below table if we analyze the column CANDYBAR_WEIGHT_OZ are all same and it is associated with only CANDYBAR_ID and not with rest of the primary key, it is neither associated with RESPONDENT_ID nor SURVEY_DATE i.e. CANDYBAR_WEIGHT_OZ associated with part of primary key. 
 ![Alt text](images/2nf.png?raw=true "2NF")
 - A table is in **Third normal form** if it is in first normal form, second normal form and all attribute must be directly associated with the primary key.
 ![Alt text](images/3nf.png?raw=true "3NF")
 - Large tables are broken into smaller tables known as **Fact table and Dimension table**. Skinny tables are known as fact table and they are the ones that are directly related with primary key like in the above table OVERALL_RATING is directly related with primary keys, dimension table are the other information table that were created due to elimination from 1NF, 2NF and 3NF. Together fact and dimension table create the original fat table.
 ![Alt text](images/FandD.png?raw=true "Fact and Dimensions")
 - SET TIMING ON is used to see the elapsed time:

```sql
SET TIMING ON
SELECT SUM(TASTE_RATING) AS TOT_TASTE 
    FROM CANDYBAR_HISTORICAL_DATA;
    
    TOT_TASTE
    27500000
    
    Elapsed: 00:00:09:993

SELECT SUM(TASTE_RATING) AS TOT_TASTE 
    FROM CANDYBAR_FACT;
    
    TOT_TASTE
    27500000
    
    Elapsed: 00:00:02:594

```
 - As you can see from above elapsed time putting historical data table in 3NF reduces elapsed time to quarter of original elapsed time that's why we should not jump to indexes.
 - A **Candidate key** is one that can identify each row of a table uniquely.
 - Generally a candidate key becomes the **primary key** of the table. If the table has more than one candidate key, one of them will become the primary key, and the rest are called **alternate keys**. 
 - A key formed by combining at least two or more columns is called **Composite key**.
 - *A primary key is a unique key and contains no null values.*
 - **Oracle automatically puts index on primary key** to help you automatically join FACT and DIMENSION Tables quicker. **Oracle doesn’t automatically index foreign key.**
 - Below example creates **inline constraint syntax** that is because we are using CONSTRAINT keyword on the same line as the definition of the column, this syntax only allows you to have single column as the primary key:
 
 ```sql
 CREATE TABLE DATE_DIM(
    SURVEY_DATE DATE CONSTRAINT pk_surveydate PRIMARY KEY,
    SURVEY_YEAR NUMBER,
    SURVEY_MONTH NUMBER);
 ```
 
 - If you need to have multiple column as primary keys than use **out of line CONSTRAINT syntax** to indicate that multiple column act as primary key:
 ```sql
  CREATE TABLE CANDYBAR_FACT(
     RESPONDENT_ID          NUMBER
     CANDYBAR_ID            NUMBER
     SURVEY_DATE            DATE
     TASTE_RATING           NUMBER
     APPEARANCE_RATING      NUMBER
     TEXTURE_RATING         NUMBER
     OVERALL_RATING         NUMBER
     LIKELIHOOD_PURCHASE    NUMBER
     NBR_CONSUMED           NUMBER
     CONSTRAINT pk_candybarfact PRIMARY KEY (RESPONDENT_ID, CANDYBAR_ID, SURVEY_DATE)
     );
  ```

 - A **Foreign Key** is a column that is the primary key of another table. So in order to create a foreign key first make it primary key in that another table. So as shown in below we have **CANDYBAR_MFR_ID** this is than used to refer as foreign key in another table as shown below:
 ```sql
 CREATE TABLE CANDYMFR_DIM(
    CANDYBAR_MFR_ID     NUMBER CONSTRAINT pk_candybarmfrid PRIMARY KEY,
    CANDYBAR_MFR_NAME   VARCHAR2(50)
 );
 
 CREATE TABLE CANDYBAR_DIM(
    CANDYBAR_ID     NUMBER CONSTRAINT pk_candybarid PRIMARY KEY,
    CANDYBAR_NAME   VARCHAR2(50),
    CANDYBAR_MFR_ID CONSTRAINT fk_candybarmfrid
                        REFERENCES CANDYBAR_MFR_ID(CANDYBAR_MFR_ID),
    CANDYBAR_WEIGHT_OZ NUMBER
 );
 ```
  here the CONSTRAINT key word gives the name to a constraint and REFERENCES says with table and ID it needs to refer to. So the foreign key is created using the CONSTRAINT keyword along with REFERENCES keyword. **Note that data type is not mentioned in the foreign key of CANDBAR_MFR_ID that is because the data type that is used is from the CANDYMFR_DIM(CANDYBAR_MFR_ID).**
  
 - One way to make SQL queries execute faster is to select only those columns that are used in analysis rather than all i.e. replace * with the name of columns you need.
 - Don't include columns in your table that will never be used. Separate out infrequently used columns into a separate table. for example in below table we have columns in below that might be rarely used these are moved to separate table (auxillary table) to make FACT table skinny:
 ![Alt text](images/AUX.png?raw=true "AUX")
 - Placing a column that contain most NULL values at the end of the table saves space. This can also make execution time faster as the armature of the read/write operation have to move less.
 ![Alt text](images/null.png?raw=true "null")

### Optimizing NON-JOINS:
 - Use SELECT to limit columns: SELECT RESPONDENT_ID, TASTE_RATING.. the more columns you bring back the more temporary space will be used.
 - Avoid using asterisk(*) SELECT * FROM .. 
 - Use WHERE to limit number of rows that are coming back from database.
 - Avoid implicit conversion, i.e. make sure that a constant that you are comparing with a column datatype matches as well, if not an implicit conversion will happen and this will slow down your query. For example comparing a String to a Number will force an implicit conversion, and also comparing a text string with a date to a column that contains a date data type will also do an implicit conversion.
 - Oracle recommends to use WHERE Clauses: Ands and Equijoins
 - WHERE and index (later explained)

#### INTERSECT vs INNER JOIN: 
 - INTERSECT is same as Set intersection.
 - INTERSECT performs better than INNER JOIN.
 - Unlike an INNER JOIN, INTERSECT performs its own DISTINCT without the need of DISTINCT keyword. 
 - Another advantage of INTERSECT is it can be on more than two tables, we just follow up with INTERSECT and another query.
 ![Alt text](images/intersect.png?raw=true "Intersect")

```sql
SELECT DISTINCT A.ID, A.DESCR 
    FROM LEFT_T A 
    INNER JOIN RIGHT_T B 
    ON A.ID = B.ID;

SELECT A.ID, A.DESCR 
    FROM LEFT_T A 
    INTERSECT 
SELECT B.ID, B.DESCR 
    FROM RIGHT_T B;

```

#### MINUS vs LEFT JOIN
 - Minus is same as set minus.
 - Minus performs better than corresponding join.
 - MINUS performs its own DISTINCT.
 - Example:
 ![Alt text](images/minus.png?raw=true "Minus")
 ![Alt text](images/minus_1.png?raw=true "Minus_1")
 
#### Corelated Sub queries:
 - In a correlated sub-query a query contains one or more columns from the outer query. Correlated subquery generally perform poorly as the database loops to the columns from outer query. 
 - wherever possible rewrite the query; for instance below query takes little over 5 seconds, this query is rewritten as INNER JOIN.
 ![Alt text](images/cor.png?raw=true "Cor")
 ![Alt text](images/cor-in.png?raw=true "Cor In")
 
#### IN Condition vs EXISTS Condition
 - IN Condition: It tests a value for membership in a list of values or subquery.
 - An EXISTS condition tests for existence of rows in a subquery. EXISTS Condition looking for atleast one row to return TRUE.
 - In both IN and EXISTS, matched inner query causes outer query to return data.
 - If inner query small as compared to outer -> **IN**
 - If inner query large as compared to outer -> **EXISTS**
 
#### Multi-Column IN Condition vs ANDs/ORs
 - Generally a multi-column in condition performance slightly faster than series of and's or or's. Below query takes just under 10 seconds.
  ![Alt text](images/and.png?raw=true "And")
 - Let's rewrite this with IN Condition, if we see below we notice a small performance gain although not significant but this multi-column in is better in readability:
  ![Alt text](images/multi-in.png?raw=true "Multi IN")

#### With Clause (Subquery factoring clause)
 - Neatens your SQL Code.
 - May significantly reduce execution time.
 - The WITH query_name clause lets you assign a name to a subquery block. You can then reference the subquery block multiple places in the query by specifying the query name. Oracle optimizes the query by treating the query name as either an inline view or as a temporary table.
 - The WITH clause query runs once and oracle than uses the same results wherever this are referred to.
 - Example below shows a query without WITH:
 ![Alt text](images/no-with.png?raw=true "No With")
 - Example below shows a query with WITH:
 ![Alt text](images/with.png?raw=true "With")
 
#### APPEND Hint for INSERTs
 - APPEND hint speeds up INSERTs in the table.
 - When you DELETE rows from the table ORACLE doesn't collapse the table to squeeze out those blank rows but when you insert new data into the table oracle will attempt to find those deleted rows and fill them in with new data and this does takes time.
 - In order to speeds up inserts use:
 ```sql
  INSERT /*+APPEND*/ -- this may look like a comment but + sign indicates to oracle that what's following is a hint.
  ```
 - APPEND will tell oracle not to search for deleted rows insert slap the new data at the end of the table.
 - It has been observed that even if you dont have deleted rows using append significantly reduces execution time.
 ![Alt text](images/append.png?raw=true "Append")

### Optimizing Joins

#### ON vs WHERE Clause
 - No significant reduction time but use ON Clause to indicate JOIN criteria and use WHERE Clause to indicate table subsetting criteria.
 ![Alt text](images/on-where.png?raw=true "On Where")

#### Order of Tables on FROM Clause
 - Tables with fewer rows first (here number of rows is actually subsetting criteria that is applied to it and not the number of rows in the table individually).
 -  Significantly reduces execution time. (but if you have indexes order of tables shouldn't matter).
 ![Alt text](images/order-from.png?raw=true "Order From")
 
#### Avoid Cartesian Products
 - All combinations of rows produced from one table against another table.
 - possible use of lot of temporary space.
 - try to avoid, if possible! 

## Overview Of Indexes

### What is an Index?
 - 

## General Questions
 - CREATE INDEX myindex oN MYTABLE(MYCOL) what type of index will be created by this statement?
non-clustered index. By default a clustered index is created only on primary key.
 - Lock escalation is the process of converting a lot of low level locks (like row locks, page locks) into higher level locks (like table locks). 
 - Difference between truncate and delete
  - truncate is a DDL and Delete is a DML
  - Delete has no autocommit, a truncate is auto commit
  - delete doesn’t recover space but truncate recover space
  - In oracle truncate cannot be rollbacked
 - What are the steps to optimise a sql query?
  - Check if the index is available
  - Full table scan
  - AWR report gives the time of the sql query or check the EXPLAIN PLAN
  - Too much normalisation
  - Rewriting the SQL statement using the IN predicate instead of the OR operator consistently and substantially improves data retrieval speed.
 - What is the impact of adding an index on performance of select/update/insert/delete query and why?
  - A general rule of thumb is that the more indexes you have on a table, the slower INSERT, UPDATE, and DELETE operations will be. This is why adding indexes for performance is a trade off, and must be balanced properly.
  - Indexes should not be used on columns that contain a high number of NULL values.
  - Indexes should not be used on small tables.
  - Columns that are frequently manipulated should not be indexed. Maintenance on the index can become excessive.
  - The insert statement is the only operation that cannot directly benefit from indexing because it has no where clause.
  - Index shouldn’t be used if the WHERE clause returns a high number of results, if a high number of data rows is returned than the database server constantly has to read the index, and then the table, and then the index, and then the table, and so on. In this case, it may be more efficient for a full table scan to occur because a high percentage of the table must be read anyway.
  - do not create an index on a column such as gender, or any column that contains very few distinct values.
  - To get a pretty good idea how your current indexes are being used, reference the sys.dm_db_index_usage_stats DMV:

```sql
select * from sys.dm_db_index_usage_stats
```
 - What is a full table scan?
  - A full table scan occurs when an index is either not used or there is no index on the table(s) being used by the SQL statement. Full table scans usually return data much slower than when an index is used.
 - Explain a sub query? How does a sub-query impact perfomance?
  - A subquery is a query embedded within the WHERE clause of another query to further restrict data returned by the query. 
 - What is an index?
  - Simply put, an index is a pointer to data in a table. An index in a database is very similar to an index in the back of a book.
  - An index is stored separately from the table for which the index was created. An index's main purpose is to improve the performance of data retrieval. Indexes can be created or dropped with no effect on the data. However, once an index is dropped, performance of data retrieval may be slowed. An index does take up physical space and often grows larger than the table itself.
 - How do index work?
  - When an index is created, it records the location of values in a table that are associated with the column that is indexed. A full table scan would occur if there were no index on the table and the same query was executed, which means that every row of data in the table would be read to retrieve information pertaining to all individuals with the name SMITH.
 - When should index be considered?
  - Unique indexes are implicitly used in conjunction with a primary key for the primary key to work. 
  - Foreign keys are also excellent candidates for an index because they are often used to join the parent table. 
  - Columns that are frequently referenced in the ORDER BY and GROUP BY clauses should be considered for indexes. 
  - Indexes should be created on columns with a high number of unique values, or columns when used as filter conditions in the WHERE clause return a low percentage of rows of data from a table.
  - There is no cut-and-dried rule for using indexes. The effective use of indexes requires a thorough knowledge of table relationships, query and transaction requirements, and the data itself.
 - How can you performance tune a database?
   - partitioning
 - How would you remove 50 million records from 50.5 million record table called Trades that has both Foreign and primary keys?
  - If I had to update millions of records I would probably opt to NOT update. I would more likely do:

```sql
CREATE TABLE new_table as select <do the update "here"> from old_table;

index new_table
grant on new table
add constraints on new_table
etc on new_table

drop table old_table
rename new_table to old_table;
```
 - slow stored procedure, how to enhance it?

 - How does a database choose which query plan to choose?

  - The EXPLAIN PLAN command is a tool to tune SQL statements. To use it you must have an explain_table generated in the user you are running the explain plan for. 
 - What is the difference between UNION and JOIN
  - A Join is used for displaying columns with the same or different names from different tables. The output displayed will have all the columns shown individually. i.e. The columns will be aligned next to each other.
  - The UNION set operator is used for combining data from two tables which have columns with the same datatype. When a UNION is performed the data from both tables will be collected in a single column having the same datatype.
 - What is the difference between UNION and UNION ALL?
  - UNION: The UNION command is used to select related information from two tables, much like the JOIN command. However, when using the UNION command all selected columns need to be of the same data type. With UNION, only distinct values are selected.
  - UNION ALL: The UNION ALL command is equal to the UNION command, except that UNION ALL selects all values.
 - The difference between UNION and UNION ALL is that UNION ALL will not eliminate duplicate rows, instead it just pulls all rows from all the tables fitting your query specifics and combines them into a table.
 - What are triggers?
  - Triggers are special kind of stored procedures that gets executed when insert, update, delete happens on the table.
 - What is the difference between primary and unique key?
  - Both primary key and unique enforce uniqueness of the column on which they are defined. But by default primary key creates a clustered index on the column, whereas unique creates a nonclustered index by default. Another major difference is that, primary key doesn't allow NULLs, but unique key allows one NULL only.
  - With a clustered index the rows are stored physically on the disk in the same order as the index. There can therefore be only one clustered index.
  - With a non clustered index there is a second list that has pointers to the physical rows. You can have many non clustered indexes, although each new index will increase the time it takes to write new records.  It is generally faster to read from a clustered index if you want to get back all the columns. You do not have to go first to the index and then to the table. Writing to a table with a clustered index can be slower, if there is a need to rearrange the data.