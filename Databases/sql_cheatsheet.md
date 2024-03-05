# SQL Cheatsheet

1. SQL Statements 			(SELECT, UPDATE, DELETE, WHERE, GROUP BY, ORDER BY, CASE, LIMIT, OFFSET, DISTINCT, Joins, Unions, Subqueries)
2. SQL Tables & Schemas 	(CREATE, DROP, INSERT INTO, ALTER, TRUNCATE, CHECK, Indexes)
3. SQL Functions 			(Mathematical Functions, Aggregate Functions, Utility Operators, Window Functions)
4. SQL CTEs 				(Common Table Expressions and Recursive CTEs)
5. SQL Views 				(Views and Materialized Views)
6. SQL Transactions 		(BEGIN, COMMIT, ROLLBACK)
7. SQL Triggers             (CREATE, MODIFY, EXECUTE, DROP)
8. SQL Migrations 			(Changing production databases with Node.js and Laravel)
9. SQL Users and Privileges (USER, ROLE, GRANT, REVOKE)
10. SQL File System 		(EXPLAIN, EXTENSION, data_directory, relkind)
11. SQL Security 			(Prepared statements in JavaScript, PHP, Python)

## 1. SQL Statements 

<details><summary>Basic query statements.</summary>

Select all columns from a table.

```sql
SELECT * FROM <table>;
```

Select specific columns from a table.

```sql
SELECT <column-1>, <column-2> FROM <table>;
```

Where clause.

```sql
SELECT * FROM <table> WHERE <condition>;
```

Update data in a table.

```sql
UPDATE <table> SET <column> = "new_data" WHERE <condition>;
```

Delete data from a table. <strong>You should always select the data you want to delete before actually deleting it.</strong>

```sql
DELETE FROM <table> WHERE <condition>;
```

</details>

<details><summary>Grouping and other keywords.</summary>

Select only the distinct data in a table.

```sql
SELECT DISTINCT <column> FROM <table>;
```

Limit how many results the query returns.

```sql
SELECT * FROM <table> LIMIT <n>
```

Skip `n` offset of rows, and start returning data after it.

```sql
SELECT * FROM <table> OFFSET <n>
```

Sort data to a defined order. Set it to descending (`DESC`) or ascending (`ASC`). ASC is the default value for the `ORDER BY` clause.

```sql
SELECT * FROM <table> ORDER BY <column> DESC;
```

Group data by some filter.

```sql
SELECT COUNT(<column-1>), <column-2> FROM <table> GROUP BY <column-2>;
```

The `WHERE` keyword cannot be used with aggregate functions. Filtering data with aggregation can be done via the `HAVING` keyword.

```sql
SELECT COUNT(<column-1>), <column-2>
FROM <table> GROUP BY <column-2>
HAVING <condition>;
```

Using conditional logic can be done via the `CASE` expression. It goes through conditions and returns a value when the first condition is met. Once a condition is true, it will stop reading and returns the result. If no conditions are true, it returns the value in the `ELSE` clause. If there is no `ELSE` and no conditions are true, it returns `NULL`.

```sql
SELECT <column-1>, <column-2>
CASE
    WHEN <condition-1> THEN <result-1>
    WHEN <condition-2> THEN <result-2>
    WHEN <condition-3> THEN <result-3>
    ELSE result
END AS <alias>
FROM <table>;
```

</details>

<details><summary>Joins and unions.</summary>

Combine the result-set of two or more `SELECT` statements.. This will only return distinct rows.

```sql
SELECT <column> FROM <table-1>
UNION ALL
SELECT <column> FROM <table-2>;
```

To allow duplicate values in a union, the `ALL` keyword should be used.

```sql
SELECT <column> FROM <table-1>
UNION ALL
SELECT <column> FROM <table-2>;
```

Return the intersection of two result-sets.

```sql
SELECT <column> FROM <table-1>
INTERSECT
SELECT <column> FROM <table-2>;
```

Subtract a result set from another result set. It returns only the rows that appear in the first result set but do not appear in the second result set.

```sql
SELECT <column> FROM <table-1>
EXCEPT
SELECT <column> FROM <table-2>;
```

Connecting tables in the same query based on shared attributes can be done via joining. The inner join returns rows that have matching values in both tables. Note: the default `JOIN` statement equals to an `INNER JOIN`.

```sql
SELECT <coloumn-1>
FROM <table-1>
INNER JOIN <table-2> ON <coloumn-2> = <coloumn-1>;
```

Left join returns all rows from the left table with corresponding rows from the right table. If there are no matching rows, `NULL` is returned as values from the second table.

```sql
SELECT <coloumn-1>
FROM <table-1>
LEFT JOIN <table-2> ON <coloumn-2> = <coloumn-1>;
```

Right join returns all rows from the right table with corresponding rows from the left table. If there are no matching rows, `NULL` is returned as values from the second table.

```sql
SELECT <coloumn-1>
FROM <table-1>
RIGHT JOIN <table-2> ON <coloumn-2> = <coloumn-1>;
```

A full join returns all roews from both tables. If there are no matching rows in the second table, `NULL` is returned.

```sql
SELECT <coloumn-1>
FROM <table-1>
FULL OUTER JOIN <table-2> ON <coloumn-2> = <coloumn-1>;
```

Cross join returns all possible combinations of rows from both tables.

```sql
SELECT <table-1.coloumn>, <table-2.coloumn>
FROM <table-1>
CROSS JOIN <table-2>;
```

Joining three tables in one query is called a three-way join.

```sql
SELECT title, name, rating
FROM reviews
JOIN books ON books.id = reviews.book_id
JOIN authors ON authors.id = reviews.reviewer_id AND authors.id = books.author_id;
```

</details>

<details><summary>Subqueries.</summary>

A subquery is a query that is nested inside another query, or inside another subquery. The simplest subquery returns exactly one column and exactly one row. It can be used with comparison operators =, <, <=, >, or >=.

```sql
SELECT name FROM city WHERE rating = (
SELECT rating FROM city WHERE name = 'Stockholm');
```

A subquery can also return multiple columns or multiple rows. Such subqueries can be used with operators `IN`, `EXISTS`, `ALL`, or `ANY`.

```sql
SELECT name FROM city WHERE country_id IN (
SELECT country_id FROM country WHERE population > 20000000);
```

```sql
SELECT (
    SELECT MAX(price) FROM phones) AS max_price, (
    SELECT MIN(price) FROM phones) AS min_price, (
    SELECT AVG(price) FROM phones) AS avg_price;
```

A correlated subquery refers to the tables introduced in the outer query. A correlated subquery depends on the outer query. It cannot be run independently from the outer query.

```sql
SELECT * FROM city AS main_city
WHERE population > (
    SELECT AVG(population) FROM city AS average_city
    WHERE average_city.country_id = main_city.country_id
);
```

</details>

## 2. SQL Tables & Schemas 

<details><summary>Working with databases.</summary>

Create a database.

```sql
CREATE DATABASE <database>;
```

Delete a database.

```sql
DROP DATABASE <database>;
```

Backup a database to disk.

```sql
BACKUP DATABASE <database>
TO DISK = 'filepath';
```

Backup only parts of the database that have changed since the last full database backup.

```sql
BACKUP DATABASE <database>
TO DISK = 'filepath'
WITH DIFFERENTIAL;
```

Show a list of all databases.

```sql
SHOW DATABASES;
```

Switch to a database.

```sql
USE <database>;
```

</details>

<details><summary>Working with schemas.</summary>

Create a schema. A schema is a list of logical structures of data, similar to separate namespaces or containers.

```sql
CREATE SCHEMA <schema>;
```

Delete a schema.

```sql
DROP SCHEMA <schema>;
```

List schemas in MySQL for the active database.

```sql
SHOW SCHEMAS;
```

List schemas in PostgreSQL for the active database.

```sql
SELECT nspname
FROM pg_catalog.pg_namespace;
```

Select a table inside a schema.

```sql
SELECT * FROM <schema>.<table>;
```

Alter a schema.

```sql
ALTER SCHEMA <schema> [RENAME TO <new-name>];
```

</details>

<details><summary>Working with tables.</summary>

Creating a table.

```sql
CREATE TABLE articles (
  id int(20) NOT NULL AUTO_INCREMENT,
  user_id int(20),
  title varchar(128) NOT NULL,
  content text NOT NULL,
  published_at datetime DEFAULT CURRENT_TIMESTAMP,
  image_url varchar(200) DEFAULT NULL,
  PRIMARY KEY (id),
  FOREIGN KEY (user_id) REFERENCES users(id);
);
```

Deleting a table.

```sql
DROP TABLE <table>;
```

Remove all data in a big table efficiently.

```sql
TRUNCATE <table>;
```

Alter a table adding foreign key constraints.

```sql
ALTER TABLE article_category
  ADD CONSTRAINT article_category_ibfk_1 FOREIGN KEY (article_id) REFERENCES articles(id) ON DELETE CASCADE ON UPDATE CASCADE,
  ADD CONSTRAINT article_category_ibfk_2 FOREIGN KEY (category_id) REFERENCES categories (id) ON DELETE CASCADE ON UPDATE CASCADE;
```

Validate data in a column using the `CHECK` constraint.

```sql
ALTER TABLE persons
ADD CONSTRAINT CHK_person_age CHECK (age >= 18); 
```

Delete a constraint.

```sql
ALTER TABLE persons
DROP CONSTRAINT CHK_person_age;
```

Insert data into a table.

```sql
INSERT INTO <table> (coloumn-1, coloumn-2, coloumn-3)
VALUES
    (value-1, value-2, value-3),
    (value-1, value-2, value-3),
    (value-1, value-2, value-3);
```

</details>

<details><summary>Working with indexes.</summary>

Create index on a coloumn. If you do not specify an index name, the standard index name template will be used which is `<table>_<coloumn>_idx`.

```sql
CREATE INDEX <index-name>
ON <table> (coloumn-name);
```

Delete an index.

```sql
ALTER TABLE <table>
DROP INDEX <index>;
```

</details>

## 3. SQL Functions

<details><summary>Aggregate functions.</summary>

AVG()

```sql

```

COUNT()

```sql

```

MAX()

```sql

```

MIN()

```sql

```

SUM()

```sql

```

</details>

<details><summary>Comparison functions.</summary>

COALESCE

```sql

```

NULLIF

```sql

```

</details>

<details><summary>Logical operators.</summary>

AND

```sql

```

IN

```sql

```

NOT IN

```sql

```

EXISTS

```sql

```

BETWEEN

```sql

```

ALL

```sql

```

ANY

```sql

```

LIKE

```sql

```

NOT LIKE

```sql

```

</details>

<details><summary>Window functions.</summary>

[This article](https://medium.com/@manutej/mastering-sql-window-functions-guide-e6dc17eb1995) was used as reference for this section.



```sql

```



```sql

```



```sql

```

</details>

## 4. SQL CTEs

<details><summary>Common Table Expressions.</summary>



```sql

```



```sql

```



```sql

```

</details>

<details><summary>Recursive Common Table Expressions.</summary>



```sql

```



```sql

```



```sql

```

</details>

## 5. SQL Views

<details><summary>Working with views.</summary>



```sql

```



```sql

```



```sql

```

</details>

<details><summary>Materialized views.</summary>



```sql

```



```sql

```



```sql

```

</details>

## 6. SQL Transactions

<details><summary>Working with transactions.</summary>

BEGIN

```sql

```

COMMIT

```sql

```

ROLLBACK

```sql

```

</details>

## 7. SQL Triggers

<details><summary>Managing triggers.</summary>

CREATE

```sql

```

MODIFY

```sql

```

EXECUTE

```sql

```

DROP

```sql

```

</details>

## 8. SQL Migrations

<details><summary>PostgreSQL migrations with Node.js.</summary>



```sql

```



```js

```



```sh

```

</details>

<details><summary>MySQL migrations with Laravel.</summary>



```sql

```



```php

```



```sh

```

</details>

## 9. SQL Users and Privileges

<details><summary>Managing users.</summary>

CREATE

```sql

```

GRANT

```sql

```

REVOKE

```sql

```

</details>

<details><summary>Managing privileges.</summary>

CREATE

```sql

```

GRANT

```sql

```

REVOKE

```sql

```

</details>

## 10. SQL File System

<details><summary>Inside the PostgreSQL file system.</summary>

EXPLAIN

```sql

```

EXPLAIN ANALYZE

```sql

```

How to find the actual binary data on the disk for a table.

```sql

```

How to find the actual binary data on the disk for an index.

```sql

```

</details>

<details><summary>Inside the MySQL file system.</summary>

EXPLAIN

```sql

```

EXPLAIN ANALYZE

```sql

```

How to find the actual binary data on the disk for a table.

```sql

```

How to find the actual binary data on the disk for an index.

```sql

```

</details>

## 11. SQL Security

<details><summary>Prepared statements in SQL.</summary>



```sql

```

</details>

<details><summary>Prepared statements with JavaScript.</summary>



```js

```



```js

```



```js

```

</details>

<details><summary>Prepared statements with PHP.</summary>



```php

```



```php

```



```php

```

</details>

<details><summary>Prepared statements with Python.</summary>



```py

```



```py

```



```py

```

</details>
