# SQL Fundamentals: Personal Learning Notes

## Introduction

I worked through the TryHackMe SQL Fundamentals room to learn how relational databases are organized and how SQL is used to retrieve and modify their data. The room introduced database concepts first, then moved into SQL syntax, database and table management, CRUD operations, clauses, operators, and functions. I followed guided examples and used the room's MySQL environment to run queries and answer practical questions.

These notes describe the concepts covered and the commands demonstrated in the room. The exercises were guided learning activities, not a professional database administration or security assessment.

## Understanding Databases

A database is an electronically stored collection of data. Applications use databases to store and retrieve information, even when the person using the application does not see the database directly. For example, a website can use a database to manage account information, while a streaming service can store information about films and use stored data to help generate recommendations.

Databases can organize information in different ways. The room introduced relational and non-relational databases. A relational database organizes data into tables. Each table contains columns that describe fields and rows that represent individual records. This structure is useful when data has a consistent format and when relationships between records need to be represented clearly.

A non-relational database does not use the same table-based relational model as its primary way of organizing data. Depending on the system, data may be represented as key-value pairs, documents, or graphs. Document databases can store records in structures that resemble JSON. This can be useful when records do not all need to follow exactly the same structure.

Relational databases are commonly associated with SQL because SQL is used to define, query, and manipulate data in them. Non-relational systems are often described as NoSQL databases, although NoSQL is a broad category and does not mean that every system has no query language or cannot use SQL-like interfaces.

## Tables, Rows, Columns, and Records

A relational database can contain many tables, with each table holding a particular kind of information. In the room's example, a table called `books` held information about books. A row represented one book record, while columns held individual fields such as the book's name, category, publication date, or description.

This distinction is important when reading a query. A row represents a record, while a column represents a particular attribute of that record. When I insert a new book into the table, I am adding a row. When I retrieve only the publication date, I am selecting a column rather than retrieving every field.

## Primary Keys and Foreign Keys

A primary key uniquely identifies each row in a table. In the room's book example, a `book_id` field could serve as the primary key. The key lets the database distinguish one record from another, including records that might otherwise contain similar information.

A primary key does not have to be a simple number, although an incrementing integer is a common design. In the example, `AUTO_INCREMENT` was used so that MySQL could assign the next number automatically when a new row was inserted. This reduces the need to choose an identifier manually for each new record.

A foreign key is a column, or a set of columns, that refers to a key in another table, usually that table's primary key. For example, the `books` table could contain an `author_id` value that refers to an author record in an `authors` table. The author identifier in the books table is a foreign key because it connects a book record to the related author record. This relationship avoids having to repeat all of an author's information in every book record.

## What SQL Does

SQL stands for Structured Query Language. I learned to think of it as the language used to communicate with a relational database. It can be used to define database structures, retrieve information, insert records, update existing data, and delete records.

A database management system, or DBMS, is the software that manages the database and provides an interface through which users and applications can work with it. MySQL is a database management system. SQL statements are sent to MySQL, which processes them and performs the requested operation against the database.

The guided room used a MySQL command-line session. The standard form of the login command is:

```bash
mysql -u root -p
```

The `-u` option specifies the username, while `root` is the username used in the room's example. The `-p` option tells the client to prompt for a password. The password is entered at the prompt and is not normally displayed as it is typed. The room supplied its own lab credentials, which should not be assumed to apply to other systems.

Once connected, I could issue SQL statements through the MySQL prompt. In the examples, statements ended with a semicolon. The semicolon marks the end of a statement for the MySQL client and is especially useful when entering queries interactively.

## SQL Keywords and Naming Conventions

The room showed SQL keywords written in uppercase, such as `SELECT`, `CREATE DATABASE`, and `SHOW TABLES`, while database and table names were generally written in lowercase. Uppercase keywords are a readability convention, not a requirement for ordinary MySQL SQL keywords. Writing `select` instead of `SELECT` generally has the same meaning.

Identifier behavior is a separate issue. Whether database and table names are case-sensitive can depend on the database system, its configuration, and the operating system. I should therefore avoid assuming that changing the capitalization of an identifier is always harmless. Consistent naming and formatting make SQL statements easier to read and maintain.

## Creating and Managing Databases

The room introduced `CREATE DATABASE` as the statement for creating a database. The example created a database named `THM_bookmarket_db`.

```sql
CREATE DATABASE THM_bookmarket_db;
```

To display the databases available in the current MySQL environment, I used:

```sql
SHOW DATABASES;
```

The `USE` statement selects the database that subsequent statements should work with. For example, the room used database names such as `THM_books` and `tools_db` in different exercises.

```sql
USE THM_books;
```

Selecting a database is important because a query that refers to a table name needs to run in the correct database unless the table is referenced with a fully qualified name.

The room also introduced `DROP DATABASE`, which removes a database.

```sql
DROP DATABASE database_name;
```

This is a destructive operation. The example above uses a placeholder name, not a database that I deleted. Before running a `DROP DATABASE` statement in any real environment, I would need to confirm the target and understand that its database contents may be removed.

## Creating and Inspecting Tables

Tables are created inside a database. A `CREATE TABLE` statement specifies a table name and defines its columns, data types, and any constraints. The room's example used a book inventory table with fields such as an integer book identifier, a book name, and a publication date.

A simplified example of the structure discussed in the room is:

```sql
CREATE TABLE book_inventory (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    book_name VARCHAR(255) NOT NULL,
    publication_date DATE
);
```

The `INT` data type stores integer values. `VARCHAR(255)` stores variable-length character data with a maximum length of 255 characters. A `DATE` field stores a date value. `NOT NULL` means a value must be supplied for that column. `PRIMARY KEY` identifies the column as the table's primary key, and `AUTO_INCREMENT` allows MySQL to generate successive integer values for new rows.

A primary key cannot contain `NULL` values, so it does not need a separate `NOT NULL` declaration in this example. The `publication_date` column is allowed to be `NULL` because the example does not declare it `NOT NULL`. That can be useful when the date is unknown, although the database design should reflect what information is genuinely optional.

To display the tables in the currently selected database, I used:

```sql
SHOW TABLES;
```

To inspect a table's structure, I used `DESCRIBE` followed by the table name.

```sql
DESCRIBE books;
```

The output shows the table's fields, data types, whether each field accepts `NULL`, key information, default values, and extra attributes. This is useful when I need to understand a table before writing a query against it.

The room also covered `ALTER TABLE`, which changes an existing table's structure. For example, adding an integer column called `page_count` can be done with:

```sql
ALTER TABLE books
ADD page_count INT;
```

The room introduced `DROP TABLE` as the statement for deleting a table.

```sql
DROP TABLE table_name;
```

Dropping a table is destructive and should not be confused with removing only a row. `DROP TABLE` removes the table itself, whereas `DELETE` can remove selected records from a table.

## CRUD Operations

CRUD stands for Create, Read, Update, and Delete. These are four common types of operation performed on stored data. In the room, I learned the SQL statements associated with each operation and followed examples using the books and hacking tools tables.

### Creating Records with INSERT

`INSERT INTO` adds new records to a table. The column names identify where the supplied values should go, and the values must correspond to the listed columns in the same order.

```sql
INSERT INTO books (id, name, publish_date, description)
VALUES (1, 'Android Security Internals', '2014-10-01', 'A book about Android security');
```

This is a syntax example based on the room's explanation, not a claim that I independently added this exact record. Text and date values are enclosed in quotes in this example, while the integer identifier is not. Specifying the column names makes the statement easier to understand and reduces dependence on the table's full column order.

### Reading Records with SELECT

`SELECT` retrieves data from a table. The asterisk in `SELECT *` means that all columns are requested.

```sql
SELECT * FROM books;
```

This query retrieves all columns for the rows returned from `books`. During the guided exercise, I used this form to inspect the book records and the entries in the `hacking_tools` table.

I can also request only the columns needed for a particular task.

```sql
SELECT name FROM books;
```

This returns the `name` column rather than every field in each matching record. Selecting only the necessary columns can make query results easier to inspect.

### Updating Records with UPDATE

`UPDATE` changes values in existing rows. The `SET` clause specifies the column and its new value, while `WHERE` identifies the rows that should be changed.

```sql
UPDATE books
SET description = 'An in-depth guide to Android security architecture'
WHERE id = 1;
```

In this example, the description is changed only for the row whose `id` equals `1`. A `WHERE` clause is particularly important in an update statement. If it is omitted, the statement can update that column in every row of the table.

### Deleting Records with DELETE

`DELETE FROM` removes rows that match a condition.

```sql
DELETE FROM books
WHERE id = 1;
```

This statement removes the row whose identifier is `1`. It does not remove the table structure itself. As with `UPDATE`, leaving out the `WHERE` clause can cause the operation to affect every row in the table, so the target condition must be checked carefully.

## SQL Clauses

A clause is a part of an SQL statement that performs a particular role. The room introduced `FROM`, `WHERE`, `DISTINCT`, `GROUP BY`, `ORDER BY`, and `HAVING`.

`FROM` identifies the table from which data is retrieved. `WHERE` filters rows according to a condition. For example, `SELECT * FROM books WHERE id = 1;` requests all columns for the row with an identifier of `1`.

`DISTINCT` removes duplicate values from the query result. For example, `SELECT DISTINCT name FROM books;` returns each different name once. This affects the displayed results, not the stored records themselves.

`GROUP BY` groups rows that share the same value in one or more columns. It is commonly used with aggregate functions such as `COUNT`. In the room's example, grouping book records by name and counting them made it possible to see that a name such as `Ethical Hacking` appeared twice, while other names appeared once.

```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name;
```

Here, `COUNT(*)` counts the rows in each group. Grouping is useful when the goal is to summarize records instead of displaying each row individually.

`ORDER BY` sorts query results by a selected column or expression. The room demonstrated ordering by a tool's name and by publication date. `ASC` sorts in ascending order, while `DESC` sorts in descending order. If no direction is specified, ascending order is commonly the default.

```sql
SELECT name
FROM hacking_tools
ORDER BY name ASC;
```

This query returns tool names in ascending order. Changing `ASC` to `DESC` reverses the sort order.

`HAVING` filters groups, often after `GROUP BY` has been applied. This is different from `WHERE`, which filters individual rows before grouping. The room also demonstrated pattern matching with `LIKE`. Pattern matching is commonly used in a `WHERE` condition when searching rows for partial text, while `HAVING` can be used when the condition is intended to filter grouped results.

For example, the following query groups records by name and retains groups whose name contains the text `hack`.

```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name
HAVING name LIKE '%hack%';
```

The percent sign is a wildcard that matches zero or more characters. As a result, `%hack%` can match text before and after `hack`. A pattern such as `%guide` matches values ending in `guide`, while `guide%` matches values beginning with `guide`.

## SQL Operators

Operators allow a query to test conditions and combine criteria. The room introduced `LIKE`, `AND`, `OR`, `NOT`, `BETWEEN`, and comparison operators.

`LIKE` is used for pattern matching. The percent sign represents any sequence of zero or more characters. If I know only part of a tool or book name, I can use a pattern rather than requiring an exact match. For example:

```sql
SELECT name
FROM books
WHERE name LIKE '%Android%';
```

This query returns book names containing `Android`. It is a search pattern, not an exact equality comparison.

`AND` requires both connected conditions to be true. `OR` requires at least one of the connected conditions to be true. For example, a query can filter for a particular category and a particular name using `AND`. This narrows the result to rows that meet both conditions.

```sql
SELECT *
FROM books
WHERE category = 'Offensive Security'
AND name = 'Bug Bounty Bootcamp';
```

`NOT` reverses a condition. For example, `WHERE category != 'Offensive Security'` selects rows whose category is not equal to that value in MySQL. A condition such as `NOT (description LIKE '%guide%')` excludes descriptions matching that pattern. Comparisons involving `NULL` require special care because `NULL` represents a missing or unknown value and is not handled like an ordinary text or numeric value.

`BETWEEN` checks whether a value falls within an inclusive range. In the room's example, an identifier between `2` and `4` includes the values `2`, `3`, and `4`, if matching records exist.

```sql
SELECT *
FROM books
WHERE id BETWEEN 2 AND 4;
```

The comparison operators include `=`, `!=`, `<`, `>`, `<=`, and `>=`. Equality checks whether a value matches the specified value. Less-than and greater-than comparisons can be used with numeric values and dates. The inclusive forms, `<=` and `>=`, include the boundary value, whereas `<` and `>` do not.

The guided questions required combining conditions to find a tool in the `multi-tool` category whose description referred to penetration testers and geeks. The approach was to filter the `hacking_tools` table by category and then apply text-pattern conditions to the description. The exercise returned `Flipper Zero`. Another query filtered by an amount greater than or equal to `300` to identify the relevant category, and a further query filtered the `Network Intelligence` category for tools with an amount below `100`. These were answers observed while completing the room's guided database exercises.

## String Functions

SQL provides functions that transform or summarize values in query results. The room introduced `CONCAT`, `GROUP_CONCAT`, `SUBSTRING`, and `LENGTH`.

`CONCAT` combines strings into one string. This can combine values from different columns with extra text supplied in the query. In the room's book example, a book name and category were joined with explanatory text to create a combined output value. The `AS` keyword assigned a label to that output column.

```sql
SELECT CONCAT(name, ' is a type of ', category, ' book') AS book_info
FROM books;
```

The result contains a derived column named `book_info`. This does not change the stored name or category in the table. It changes how the query presents the selected data.

`GROUP_CONCAT` combines values from multiple rows into a single string for each group. The room used it to display book names grouped by category and separated by a chosen delimiter. The separator is supplied to the function, allowing the result to be easier to read.

```sql
SELECT category,
       GROUP_CONCAT(name SEPARATOR ', ') AS books
FROM books
GROUP BY category;
```

This query returns each category with its book names joined into one value. It combines the grouping behavior of `GROUP BY` with the string aggregation provided by `GROUP_CONCAT`.

`SUBSTRING` extracts part of a string. The room used this idea to retrieve the year portion of a publication date rather than displaying the entire date. For a date represented as `2021-11-15`, the first four characters are `2021`.

```sql
SELECT SUBSTRING(publish_date, 1, 4) AS published_year
FROM books;
```

This example illustrates extracting the first four characters from a value represented as text. If the field is stored as a native date, a date function such as `YEAR(publish_date)` is generally a clearer way to retrieve its year in MySQL.

`LENGTH` returns the length of a string in bytes in MySQL. For ordinary ASCII tool names, this usually corresponds to the number of characters, but the two measures can differ for multibyte text. The room used length to compare tool names and identify the longest one. Selecting both the name and its length makes the result easier to interpret.

```sql
SELECT name, LENGTH(name) AS name_length
FROM hacking_tools
ORDER BY LENGTH(name) DESC;
```

This sorts the tool names from longest to shortest according to the byte length returned by `LENGTH`. If only one longest result is needed, a query can use a limit, but tied values and the exact database behavior should be considered.

## Aggregate Functions

Aggregate functions calculate a result from multiple rows. The room introduced `COUNT`, `SUM`, `MAX`, and `MIN`.

`COUNT(*)` returns the number of rows in the result set or in each group. It is useful when I need a record count instead of the records themselves.

```sql
SELECT COUNT(*)
FROM books;
```

`SUM` adds the values in a numeric column. In the tools exercise, the `amount` field was summed to calculate the total amount across the rows in `hacking_tools`.

```sql
SELECT SUM(amount)
FROM hacking_tools;
```

The guided exercise returned a total of `1444` for that dataset. This value belongs to the room's provided data and should not be treated as a general property of the tools.

`MAX` returns the maximum value in a column, while `MIN` returns the minimum value. When applied to a publication-date column, they can identify the latest and earliest dates, respectively.

```sql
SELECT MAX(publish_date)
FROM books;

SELECT MIN(publish_date)
FROM books;
```

Aggregate functions can be combined with `GROUP BY` to calculate a separate result for each group. For example, `COUNT(*)` with `GROUP BY name` counts the rows associated with each name rather than counting the entire table as one total.

## Practical Learning from the Room

During the guided exercises, I used the MySQL interface to inspect databases and tables, select records, sort tool names, filter rows using conditions, and apply functions to the supplied data. I used `SHOW DATABASES` to list databases, `USE` to select a database, `SHOW TABLES` to inspect its tables, and `DESCRIBE` to examine a table's structure.

The room's `tools_db` database contained a `hacking_tools` table. I used `SELECT` queries to inspect the table and combined filters to answer questions about tool categories, descriptions, and amounts. Sorting by name in ascending and descending order helped identify the first tool in each order. Pattern matching and comparison operators helped narrow the results to rows that met the conditions in each exercise.

I also learned how string and aggregate functions can reduce the amount of manual inspection required. Instead of counting rows myself, I could use `COUNT`. Instead of adding numeric values manually, I could use `SUM`. `GROUP_CONCAT` showed how multiple values could be combined into one result for each category, and `LENGTH` helped compare the lengths of tool names.

The SQL environment and data were provided as part of the guided TryHackMe room. I did not document an independent database deployment, a production database change, or a real-world SQL injection test, so these notes do not claim that I performed those activities.

## What I Learned

The main lesson was that SQL statements become easier to understand when I identify the role of each part. `SELECT` specifies what data I want to retrieve, `FROM` specifies where it comes from, and `WHERE` limits the rows that match a condition. Clauses such as `GROUP BY`, `ORDER BY`, and `HAVING` change how results are grouped, sorted, or filtered. Operators allow me to combine conditions, while functions calculate or transform values in the results.

I also learned that database structure matters. Rows represent records, columns represent fields, primary keys identify records uniquely, and foreign keys connect related tables. Understanding this structure makes it easier to write queries that retrieve the intended information.

Finally, I need to be careful with statements that modify data or database structure. `UPDATE`, `DELETE`, `DROP TABLE`, and `DROP DATABASE` can make significant changes. A correct condition and a clear understanding of the target are essential before running destructive statements.

## Conclusion

The SQL Fundamentals room gave me an introduction to relational databases and the SQL statements used to work with them. I practiced reading the structure of a database, retrieving and filtering records, sorting results, combining conditions, and using string and aggregate functions against the room's supplied data. I now have a foundation for reading basic SQL queries and building them step by step. I still need more practice writing queries independently and checking their results carefully before I can be confident using SQL beyond guided exercises.
