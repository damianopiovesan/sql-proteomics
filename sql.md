Databases using SQL
===================

Setup
-----

1. Install Firefox
2. Install the SQLite Manager add on **Tools -> Add-ons -> Search -> SQLite
Manager -> Install -> Restart**
3. Download the [Portal Database](../../data/biology/human_atlas/human_atlas.sqlite)
4. Download the [Mapping File](../../data/biology/human_atlas/hgnc.csv)
5. Open SQLite Manage **Firefox Button -> Web Developer -> SQLite Manager**


Relational databases
--------------------

* Relational databases store data in tables with fields (columns) and records
  (rows)
* Data in tables has types, just like in Python, and all values in a field have
  the same type ([list of data types](#datatypes))
* Queries let us look up data or make calculations based on columns
* The queries are distinct from the data, so if we change the data we can just
  rerun the query


Database Management Systems
---------------------------

There are a number of different database management systems for working with relational
data. We're going to use SQLite today, but basically everything we teach you
will apply to the other database systems as well (e.g., MySQL, PostgreSQL, MS
Access, Filemaker Pro). The only things that will differ are the details of
exactly how to import and export data and the [details of data types](#datatypediffs).


The data
--------

This is data on the human proteome expression level in healty cerebral cortex and
head/neck cancer tissues. This is part of the Human Protein Atlas Project that
studied the expression level on 44 tissue types from 144 healty individuals 
and other 20 from 216 cancer patients. Experiments are based on immunohistochemical 
staining using tissue micro arrays.

This is a real dataset that has been used in over 300 publications. I've
simplified it removing most of the data for the workshop, but you can download the
[full dataset](http://www.proteinatlas.org/about/download) ("Normal tissue data", 
"Cancer tumor data") and work with it using exactly the same tools we'll learn about today.


Database Design
---------------

1. Order doesn't matter
2. Every row-column combination contains a single *atomic* value, i.e., not
   containing parts we might want to work with separately.
3. One field per type of information
4. No redundant information
     * Split into separate tables with one table per class of information
	 * Needs an identifier in common between tables – shared column - to
       reconnect (foreign key).


Import
------

1. Start a New Database **Database -> New Database**
2. Start the import **Database -> Import**
3. Select the file to import
4. Give the table a name (or use the default)
5. If the first row has column headings, check the appropriate box
6. Make sure the delimiter and quotation options are correct
7. Press **OK**
8. When asked if you want to modify the table, click **OK**
9. Set the data types for each field

***EXERCISE: Import HGNC gene symbol mapping file (hgnc.csv)***

You can also use this same approach to append new data to an existing table.

---
layout: lesson
root: ../..
title: "sql-human_atlas"
---

Contributors: Damiano Piovesan

---


Basic queries
-------------
Let's start by using the **tissue** table.


Here we have data on expression level for genes in the cerebrabl cortex tissue,
including cell type, expression level, expression type, and data reliability.

Let’s write an SQL query that selects only the cell type column from the tissue
table:

    SELECT "Cell type" FROM tissue;

We have capitalized the words SELECT and FROM because they are SQL keywords.
SQL is case insensitive, but it helps for readability – good style. Moreover, we 
have quoted the "Cell type" column name since the space in column name 
is misintrepeted by the SQL language. 

If we want more information, we can just add a new column to the list of fields,
right after SELECT:

    SELECT Gene, "Cell type", Level FROM tissue;

Or we can select all of the columns in a table using the wildcard *:

    SELECT * FROM tissue;

### Unique values

If we want only the unique values so that we can quickly see what species have
been sampled we use ``DISTINCT``:

    SELECT DISTINCT "Cell type" FROM tissue;

If we select more than one column, then the distinct pairs of values are
returned:

    SELECT DISTINCT "Cell type", Level FROM tissue;

### Calculated values

We can also do calculations with the values in a query. 
For example, let's switch to the cancer table. 
If we wanted to look at the number of affected individual
for different expression levels and genes, but we needed it as a fraction of the 
total patients. We would use:

    SELECT Gene, Level, "Expression type", "Count patients" / "Total patients" from cancer;

When we run the query, the expression ``"Count patients" / "Total patients"`` is evaluated for each row
and appended to that row, in a new column. However, when computing operations on columns of type
"integer" we will get "integer" output. To get decimal digits we "cast" (convert) one of the "integer" 
columns to "float" (real numbers). This is done implicitly by appending 0.0:

	SELECT Gene, Level, "Expression type", ( "Count patients" + 0.0 ) / "Total patients" from cancer;

Expressions can use any fields, any arithmetic operators (+ - * /) and a 
variety of built-in functions (). For example, we could round the values 
to make them easier to read.

	SELECT Gene, Level, "Expression type", ROUND(( "Count patients" + 0.0 ) / "Total patients", 2) from cancer;

***EXERCISE: Write a query that returns
             The Gene, Level, Total patients and the fraction of affected patients 
			 for different expression levels in percentage***

Filtering
---------

Databases can also filter data – selecting only the data meeting certain
criteria.  For example, let’s say we only want data for highly expressed genes.
We need to add a WHERE clause to our query:

    SELECT * from cancer WHERE Level = "High";

We can do the same thing with numbers.
Here, we only want the data for genes having at least 1 affected patient:

    SELECT * FROM cancer WHERE "Count patients" >= 1;

We can use more sophisticated conditions by combining tests with AND and OR.
For example, suppose we want the data for genes having at least 1 affected patient and
highly expressed:

	SELECT * FROM cancer WHERE ("Count patients" >= 1) AND (Level = "High");

Note that the parentheses are not needed, but again, they help with readability.
They also ensure that the computer combines AND and OR in the way that we
intend.

If we wanted to get data for any of the genes,
which have expression level High, Medium and Low 
we could combine the tests using OR:

	SELECT * FROM cancer WHERE (Level = "High") OR (Level = "Medium") OR (Level = "Low");

***EXERCISE: Write a query that returns
	The Gene, Level, Total patients caugth on 
	cancer table that have at least 50% of affected
	patients***


Saving & Exporting queries
--------------------------

* Exporting:  **Actions** button and choosing **Save Result to File**.
* Save: **View** drop down and **Create View**


Building more complex queries
-----------------------------

Now, lets combine the above queries to get data for the 3 different expression levels 
with at least 1 affected patien. This time, let’s use IN as one way to make the query easier
to understand.  It is equivalent to saying 
``WHERE (Level = "High") OR (Level = "Medium") OR (Level = "Low")``, but reads more neatly:

    SELECT * FROM cancer WHERE ("Count patients" >= 1) AND (Level IN ("High","Medium","Low"));

    SELECT * 
	FROM cancer 
	WHERE ("Count patients" >= 1) AND (Level IN ("High","Medium","Low"));

We started with something simple, then added more clauses one by one, testing
their effects as we went along.  For complex queries, this is a good strategy,
to make sure you are getting what you want.  Sometimes it might help to take a
subset of the data that you can easily see in a temporary database to practice
your queries on before working on a larger or more complicated database.


Sorting
-------

We can also sort the results of our queries by using ORDER BY.

	SELECT * FROM cancer ORDER BY "Count patients" ASC;

The keyword ASC tells us to order it in Ascending order.
We could alternately use DESC to get descending order.

    SELECT * FROM cancer ORDER BY "Count patients" DESC;

ASC is the default.

We can also sort on several fields at once.
We might want to order by cell type then expression level.

    SELECT * FROM cancer ORDER BY "Total patients","Count patients" DESC;

***Exercise: Write a query that returns
      		 The Gene, Level, Total patients from the 
			 cancer table, sorted with the largest 
			 fraction (ratio) of affected patients at the top***


Order of execution
------------------

Another note for ordering. We don’t actually have to display a column to sort by
it. For example, let’s say we want to order by the "Count patients", but we only want
to see gene and expression level.

    SELECT Gene, Level FROM cancer ORDER BY "Count patients";

We can do this because sorting occurs earlier in the computational pipeline than
field selection.

The computer is basically doing this:

1. Filtering rows according to WHERE
2. Sorting results according to ORDER BY
3. Displaying requested columns or expressions.


Order of clauses
----------------
The order of the clauses when we write a query is dictated by SQL: SELECT, FROM, WHERE, ORDER BY
and we often write each of them on their own line for readability.


***Exercise: Let's try to combine what we've learned so far in a single query.
Using the cancer table write a query to display the gene ID, expression level, 
and the percentage of affected patients (rounded to two decimal places), for expression levels 
with more than 50% of affected patients, ordered alphabetically by gene ID.***


**BREAK**

Aggregation
-----------

Aggregation allows us to combine results by grouping records based on value and
calculating combined values in groups.

Let’s go to the cancer table and find out how many expression levels have been tested
in the experiment. Using the wildcard simply counts the number of records (rows)

    SELECT COUNT(*) FROM cancer;

We can also find out how many patients are affected.

    SELECT COUNT(*), SUM("Count patients") FROM cancer;

***Do you think you could output this value as a fraction of total patients, rounded to 3 decimal
   places?***

    SELECT ROUND( (SUM("Count patients") + 0.0 ) / SUM("Total patients"), 3) FROM cancer;

There are many other aggregate functions included in SQL including
MAX, MIN, and AVG.

***From the surveys table, can we use one query to output the total number of affected patients,
   the average number, and the min and max number?***

Now, let's use the tissue table and see how many genes were counted for each cell type. 
We do this using a GROUP BY clause

	SELECT "Cell type", COUNT(Gene)
    FROM tissue
    GROUP BY "Cell type"

The query becames more interesting when filtering for different expression levels:

	SELECT "Cell type", COUNT(Gene)
    FROM tissue
	WHERE Level = "High"
	GROUP BY "Cell type"

GROUP BY tells SQL what field or fields we want to use to aggregate the data.
If we want to group by multiple fields, we give GROUP BY a comma separated list.


***EXERCISE: Write queries that return:***
***1. How many genes were counted for each cell type and each expression level***

***2. Consider cancer table. How many genes are expressed at different levels and
	  how many evidecences (Count patients) are available?***


We can order the results of our aggregation by a specific column, including the
aggregated column. Let’s count the number of genes of each cell type and each expression
level, ordered by the count:

	SELECT "Cell type", Level, COUNT(Gene)
    FROM tissue
	GROUP BY "Cell type", Level
	ORDER BY COUNT(Gene)


SQL allows the use of aliases to simplify the query. Let's us take the cancer table and
count genes for different ratio of affected patients and different expression levels.
We aggregate by expression level and ratio and order the result by expression level, ratio and 
gene count: 

	SELECT Level, ROUND(("Count patients" + 0.0 ) / "Total patients",2) AS Ratio ,COUNT(Gene)
    FROM cancer
	GROUP BY Level, Ratio
	ORDER BY Level, Ratio, COUNT(Gene)

The "AS" keyword allows to output a new column with an alternative alias name (Ratio) and use
it inside the same query statement. 



Joins
-----

To combine data from two tables we use the SQL JOIN command, which comes after
the FROM command.

We also need to tell the computer which columns provide the link between the two
tables using the word ON. What we want is to join the data with the same
gene codes.

    SELECT *
    FROM tissue
    JOIN cancer ON tissue.Gene = cancer.Gene

ON is like WHERE, it filters things out according to a test condition.  We use
the table.colname format to tell the manager what column in which table we are
referring to.

We often won't want all of the fields from both tables, so anywhere we would
have used a field name in a non-join query, we can use *table.colname*

For example, what if we wanted information on expressed genes in healty 
individials and in head/neck cancer patients at the same time, but we want
to see only some useful columns.

	SELECT tissue.Gene, tissue."Cell type", tissue.Level, cancer.Level
	FROM tissue
	JOIN cancer ON tissue.Gene = cancer.Gene


***Exercise: Write a query that returns the gene, the cell type, the expression
	levels for both healty and sick individuals, as well as the number of affected patients***

Joins can be combined with sorting, filtering, and aggregation. So, if we
wanted to see genes that are normally low expressed in healty individuals
but highly expressed in cancer patients, we could do something like:


	SELECT 
		tissue.Gene, 
		tissue."Cell type", 
		tissue.level AS "Healty Level", 
		( (cancer."Count patients" + 0.0) / cancer."Total patients" ) as Ratio
	FROM 
		tissue
		JOIN cancer ON tissue.Gene = cancer.Gene
	WHERE 
		tissue.Level IN ("Low", "Not detected") AND cancer.Level = "High" AND Ratio  > 0.5
	ORDER BY 
		"Cell type", Ratio DESC


Moreover, to make the gene ID more "human understandable" we can use the imported HGNC 
mapping table and do a 3 table join. 
If the name of the new table is "hgnc" and columns are "Gene" and "Symbol", we could do:

	
	SELECT 
		tissue.Gene, 
		hgnc.Symbol, 
		tissue."Cell type", 
		tissue.level AS "Healty Level", 
		( (cancer."Count patients" + 0.0) / cancer."Total patients" ) as Ratio
	FROM 
		tissue 
		JOIN cancer ON tissue.Gene = cancer.Gene 
		JOIN hgnc ON hgnc.Gene = cancer.Gene
	WHERE 
		tissue.Level IN ("Low", "Not detected") AND cancer.Level = "High" AND Ratio  > 0.5
	ORDER BY 
		"Cell type", Ratio DESC


Adding data to existing tables
------------------------------

* Browse & Search -> Add
* Enter data into a csv file and append


Other database management systems
---------------------------------

* Access or Filemaker Pro
    * GUI
    * Forms w/QAQC
	* But not cross-platform
* MySQL/PostgreSQL
    * Multiple simultaneous users
	* More difficult to setup and maintain


Q & A on Database Design (review if time)
-----------------------------------------

1. Order doesn't matter
2. Every row-column combination contains a single *atomic* value, i.e., not
   containing parts we might want to work with separately.
3. One field per type of information
4. No redundant information
     * Split into separate tables with one table per class of information
	 * Needs an identifier in common between tables – shared column - to
       reconnect (foreign key).


<a name="datatypes"></a> Data types
-----------------------------------

| Data type  | Description |
| :------------- | :------------- |
| CHARACTER(n)  | Character string. Fixed-length n  |
| VARCHAR(n) or CHARACTER VARYING(n) |	Character string. Variable length. Maximum length n |
| BINARY(n) |	Binary string. Fixed-length n |
| BOOLEAN	| Stores TRUE or FALSE values |
| VARBINARY(n) or BINARY VARYING(n) |	Binary string. Variable length. Maximum length n |
| INTEGER(p) |	Integer numerical (no decimal). |
| SMALLINT | 	Integer numerical (no decimal). |
| INTEGER |	Integer numerical (no decimal). |
| BIGINT |	Integer numerical (no decimal). |
| DECIMAL(p,s) |	Exact numerical, precision p, scale s. |
| NUMERIC(p,s) |	Exact numerical, precision p, scale s. (Same as DECIMAL) |
| FLOAT(p) |	Approximate numerical, mantissa precision p. A floating number in base 10 exponential notation. |
| REAL |	Approximate numerical |
| FLOAT |	Approximate numerical |
| DOUBLE PRECISION |	Approximate numerical |
| DATE |	Stores year, month, and day values |
| TIME |	Stores hour, minute, and second values |
| TIMESTAMP |	Stores year, month, day, hour, minute, and second values |
| INTERVAL |	Composed of a number of integer fields, representing a period of time, depending on the type of interval |
| ARRAY |	A set-length and ordered collection of elements |
| MULTISET | 	A variable-length and unordered collection of elements |
| XML |	Stores XML data |


<a name="datatypediffs"></a> SQL Data Type Quick Reference
----------------------------------------------------------

Different databases offer different choices for the data type definition.

The following table shows some of the common names of data types between the various database platforms:

| Data type |	Access |	SQLServer |	Oracle | MySQL | PostgreSQL |
| :------------- | :------------- | :---------------- | :----------------| :----------------| :---------------|
| boolean	| Yes/No |	Bit |	Byte |	N/A	| Boolean |
| integer	| Number (integer) | Int |	Number | Int / Integer	| Int / Integer |
| float	| Number (single)	|Float / Real |	Number |	Float |	Numeric
| currency | Currency |	Money |	N/A |	N/A	| Money |
| string (fixed) | N/A | Char |	Char | Char |	Char |
| string (variable)	| Text (<256) / Memo (65k+)	| Varchar |	Varchar / Varchar2 |	Varchar |	Varchar |
| binary object	OLE Object Memo	Binary (fixed up to 8K) | Varbinary (<8K) | Image (<2GB)	Long | Raw	Blob | Text	Binary | Varbinary |
