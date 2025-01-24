# SQL vs. SQLite

[*Note: You can skip this section for now and come back to it while you're doing the tasks in Project 1.*]

## Why Are We Using SQLite in This Class?

As you may have learned mostly SQL synax, it will not be the engine that we use for this project. Instead, we will use a more lightweight variant called SQLite. As noted on the docs of [SQLite](https://www.sqlite.org/whentouse.html) official website, *Client/server SQL database engines strive to implement a shared repository of enterprise data. They emphasize scalability, concurrency, centralization, and control. SQLite strives to provide local data storage for individual applications and devices.* As such, SQLite is very easy to set up and run, while a standard SQL engine requires setting up an entire server. 

Now, with downloading an app of several megabytes, you can quickly run SQL-like queries on any database you want!

## New Autograder

Starting this semester, we will be using a new autograder integrating [Cosette](https://cosette.cs.washington.edu/) to grade your work. The Cosette SQL Solver will check the equivalence of two SQL queries, and that implies if you are not writing in standard SQL syntax, but somehow SQLite engine understood it, Cosette will complain. And you will be deducted 5% of points for that question even if the output produced by your query matches the output of the official solution. 

## SQLite Syntax Difference

SQLite is a much more tolerant language than SQL, so a lot of queries that raise an error in SQL will be inferred and run successfully by SQLite. We do not wish that you utilize this tolerance to write "incorrect" queries. Next, we will go over some most common errors that students make and which Cosette Solver will complain about.

Specifically: 
* There is support for `LEFT OUTER JOIN` but not `RIGHT OUTER` or `FULL OUTER`.
  * To get equivalent output to `RIGHT OUTER` you can reverse the order of the tables (i.e. `A RIGHT JOIN B` is the same as `B LEFT JOIN A`.
  * While it isn't required to complete this assignment, the equivalent to `FULL OUTER JOIN` can be done by `UNION`ing `RIGHT OUTER` and `LEFT OUTER`
* There is no regex match (`~`) tilde operator. You can use `LIKE` instead.
* There is no `ANY` or `ALL` operator.

## Most Common SQL Errors
- Use the alias directly in `WHERE/HAVING` clause
  ``` sql
  SELECT birthyear, AGG(col1) AS foo, ...
    FROM 186_TAs
    GROUP BY birthyear
    HAVING foo > "bar"
    ...
  ```

  The problem here is that SELECT is applied after the TAs are "GROUP BY"ed and filtered by "HAVING". At the stage of "HAVING", the SQL engine doesn't understand the alias "foo" in the SELECT clause yet. 

- "=="
  
  Something that you may learn in the first day of a CS class includes that computers start at index 0, and "=" is the assignment operator rather than comparison. This convention will be broken in SQL world, where you should use "=" for direct comparison. 

- `(INNER | { LEFT | RIGHT | FULL } [OUTER]) JOIN` without a join condition

  In SQL, only `NATURAL JOIN` does not require a join condition as it automatically infers the common column names. It is the language's rule that you are required to give some condition with the `ON` clause.

- `GROUP BY` without aggregate

  This is probably one of the most common mistakes made by using SQLite. Let's take a look at the following example, where we are trying to gain insight into the attendance rate of each student in 186, displayed with their sid, number of appearances in sections, along with their names.

  ``` sql
  SELECT s.sid, SUM(a.attendance) AS attend_rate, s.name
    FROM 186_students s INNER JOIN section_attendance a ON s.sid = a.sid
    GROUP BY s.sid
    ...
  ```
  In this SQL query, `s.sid` will be recognized without any issue, as it's the GROUP BY key; same for `SUM(a.attendance)`, as it is the Aggregate column. But how about `s.name`? It doesn't fall into either of the categories, so it is invalid to use it here.

## OK, so SQLite Seems Untrustworthy...

Now, you may be very concerned that some code that gets executed in SQLite engine will fail the autograder check. Don't worry about that, as SQLite is a commercial use database engine, it is quite fault-tolerant, that is to say, it catches a lot of syntax issues. The ones mentioned above are just slightly more demanding syntax rules in SQL. So if your code passes the SQLite check, and it is following the rule taught in class, you should be good to go!