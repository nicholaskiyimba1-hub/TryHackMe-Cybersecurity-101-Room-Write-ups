# TryHackMe SQLMap: The Basics

## Introduction

In this room, I studied how SQLMap automates the detection and exploitation of SQL injection vulnerabilities in web applications. The exercises connected the relationship between a browser, a web server, and a database with the way an application can become vulnerable when it places untrusted input directly into an SQL query.

The practical work took place in the TryHackMe lab. SQL injection testing should only be performed against systems I own or have explicit permission to assess. These notes describe the room's controlled exercise rather than authorization to test unrelated websites.

## How a Web Application Communicates with a Database

When I submit a login form, the browser sends an HTTP request to the web server. The application processes the submitted username and password and may query a database to determine whether the credentials match a stored account. The database returns the query result to the application, and the application responds to the browser.

A request can send parameters in the URL query string or in the request body. A GET request commonly places parameters in the URL, while a POST request commonly sends form data in the request body. These are common patterns, not absolute rules. Sensitive login credentials should not be placed in a URL because URLs can be retained in browser history, server logs, proxy logs, and other systems. HTTPS protects data in transit, but it does not remove the exposure that can come from putting secrets in URLs.

SQL is the language used to query and manipulate relational databases. It is not itself the language that connects every website to its database. The web application, written in a server-side language such as Python or PHP, uses a database driver or library to communicate with the database by sending SQL statements and parameter values.

## Parameterized Queries and SQL Injection

A secure application uses parameterized queries, also called prepared statements where supported. The SQL statement is kept separate from the values supplied by the user. A placeholder such as a question mark represents a parameter, and the database driver passes the username and password as data. SQL keywords or quotation marks in a parameter value are therefore not interpreted as part of the SQL statement merely because they appear in user input.

An unsafe application may build a query by concatenating user input into the SQL statement or by formatting input directly into a query string. If an attacker can change the structure of that query, the application may execute unintended SQL. This weakness is called SQL injection.

A classic example uses a condition that is always true, such as `1=1`, together with a comment marker to alter the logic of a vulnerable query. The exact effect depends on the original query, the database engine, the input context, and how the application constructs the statement. A successful injection does not automatically reveal every record or every password. The data that can be accessed depends on the query, database permissions, application behavior, and the type of injection.

Input validation and sanitization can help with application security, but they are not substitutes for parameterized queries. Developers should use parameterized queries for values and allowlist validation for input that must follow a particular format. Where SQL identifiers such as table names cannot be parameterized, the application should select them from a strict allowlist rather than insert arbitrary user input into the query.

## SQLMap Overview

SQLMap is an automated tool for testing SQL injection and, where a vulnerability is present and permissions allow it, interacting with the affected database. It can test parameters, identify the database management system, enumerate databases and tables, list columns, and retrieve records.

The main command starts the tool:

```bash
sqlmap
```

The `-u` option supplies a target URL. When testing a query-string parameter, the URL should include the parameter that is part of the authorized test. Quoting the URL in the shell helps prevent special characters such as ampersands from being interpreted by the shell rather than passed to SQLMap.

```bash
sqlmap -u 'https://authorized-lab.example/page.php?item=1'
```

The example URL is illustrative and is not a real target.

The `--dbs` option asks SQLMap to enumerate available databases. The `-D` option selects a database for a later operation. The `--tables` option enumerates tables, and `-T` selects a table. The `--columns` option lists the columns in a selected table. The `--dump` option retrieves records from the selected table, subject to what the database account and the vulnerability permit. The `--dump-all` option attempts to retrieve data from all accessible tables and databases, so it is generally less precise and can generate much more traffic than a targeted query. The `-h` or `--help` options display help information.

A typical sequence in an authorized lab is to identify databases first, select the database of interest, enumerate its tables, and then inspect a specific table only when the exercise requires it. This staged approach limits unnecessary requests and makes the results easier to interpret. It also avoids the assumption that every database or table is relevant to the assessment.

## Testing a Captured HTTP Request

SQLMap can use a saved HTTP request instead of a URL. The `-r` option specifies a file containing the request. This is useful when a request contains POST parameters, cookies, headers, or other details that are not represented by a simple URL.

```bash
sqlmap -r request.txt --dbs
```

In a controlled assessment, I can capture a request using browser developer tools or an authorized intercepting proxy, save the complete request to a text file, and provide that file to SQLMap. The request must be captured from the permitted target and should not contain real users' credentials or session tokens unless their use is explicitly authorized and safely controlled.

Browser developer tools can help me inspect requests by opening the Network panel, submitting a form in the lab, and selecting the request to review its URL, method, parameters, and response. Capturing the request is useful because the parameter that needs testing may be part of the URL, a form body, a cookie, or another request component.

## Enumerating Databases and Tables

The room demonstrates a step-by-step workflow. First, SQLMap is run against the lab URL with `--dbs` to enumerate the databases. Once the relevant database is known, the `-D` option selects it and `--tables` enumerates the tables within it.

An example command structure is:

```bash
sqlmap -u 'LAB_URL_WITH_PARAMETERS' --dbs
```

After identifying a database, the next command structure is:

```bash
sqlmap -u 'LAB_URL_WITH_PARAMETERS' -D 'DATABASE_NAME' --tables
```

After identifying the required table, SQLMap can list its columns:

```bash
sqlmap -u 'LAB_URL_WITH_PARAMETERS' -D 'DATABASE_NAME' -T 'TABLE_NAME' --columns
```

If the authorized exercise specifically requires retrieving the records in that table, the structure is:

```bash
sqlmap -u 'LAB_URL_WITH_PARAMETERS' -D 'DATABASE_NAME' -T 'TABLE_NAME' --dump
```

The placeholders in these examples must be replaced with values from the authorized lab. They are not intended to be run against an arbitrary website.

## SQLMap Prompts and Scan Settings

During the practical exercise, SQLMap displayed interactive prompts about the suspected database type, additional tests, possible injection techniques, and whether it should continue testing other parameters. The recommended response can be a useful starting point in a guided lab, but I should read each prompt rather than automatically accepting every default. A default is not a guarantee that a test is harmless or appropriate for every environment.

SQLMap's `--level` option controls the breadth of tests and parameters it considers, while `--risk` controls the risk level of certain tests. Higher values can lead to additional requests and potentially more intrusive tests. A low risk setting does not guarantee that a scan is undetectable or incapable of causing side effects. Testing should be scoped, authorized, and coordinated with the system owner.

The tool may also ask about testing UNION-based injection, trying alternative values, or optimizing handling of response delays. These choices relate to the techniques SQLMap uses to validate a suspected injection point and retrieve results. The correct choice depends on the target, the test scope, and the impact that additional requests could have. A finding should be reviewed and validated rather than treated as conclusive simply because a tool printed a message.

SQLMap can generate a noticeable volume of requests. Broad enumeration and dumping can be slow, noisy, and expose more data than is needed. A targeted workflow is preferable: identify the injection point, enumerate only the relevant database, inspect the required table, and retrieve only the data explicitly needed for the lab or assessment.

## Practical Exercise

In the TryHackMe practical task, I opened the provided lab login page and used the browser's developer tools to inspect the request. After submitting test input, I used the Network panel to identify the GET request and copied the URL containing the parameters used by the exercise.

I then ran SQLMap against that lab URL and requested database enumeration with `--dbs`. SQLMap tested the relevant parameter and reported six databases in the lab application. I used the `-D` option with the database named `AI` and `--tables` to enumerate its tables. The table returned by the exercise was named `user`.

I then selected the `user` table with `-T user` and used `--dump` to retrieve the lab records required by the question. The exercise asked for the password associated with the `test.com` email address, and the lab result showed `12345678`. This value is recorded here only as the answer observed in the deliberately vulnerable training environment. It is not a credential to reuse elsewhere.

The virtual machine failed during one of the enumeration steps, so I restarted it and repeated the necessary operation. This was a reminder that lab environments and virtual machines can fail, and that it is useful to understand the sequence of commands rather than depend entirely on a single uninterrupted run.

## Defensive Lessons

The root cause of SQL injection is unsafe handling of input when an application constructs SQL statements. Parameterized queries are the primary defense for user-supplied values. Applications should also use least-privilege database accounts, validate input according to expected formats, avoid exposing detailed database errors to users, and keep database software and application dependencies maintained.

Passwords should not be stored in plaintext. Applications should store passwords using a purpose-built password-hashing algorithm with appropriate salts and work factors. If an assessment finds exposed passwords, the issue should be handled as a security incident and remediated, not used to access other accounts.

Security testing tools can help identify weaknesses, but they do not replace code review, secure development practices, and careful interpretation of results. SQLMap automates many testing steps, while an understanding of HTTP requests, SQL query structure, database behavior, and the application's input handling helps me understand what its findings mean.

## Conclusion

This room introduced SQL injection, explained why parameterized queries prevent user input from changing SQL query structure, and demonstrated how SQLMap can automate testing and database enumeration in a controlled lab. I practiced identifying a request in browser developer tools, enumerating databases and tables, and retrieving the specific record required by the exercise.

The main lesson is that SQLMap is easier to use responsibly when I understand the underlying web request and SQL query. In future authorized assessments, I should keep testing within scope, use targeted enumeration, minimize data retrieval, and report the vulnerability with enough evidence for the application owner to fix it.
