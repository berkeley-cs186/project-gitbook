# Part 2: Query Optimization

![Dataspace](../../.gitbook/assets/dataspace.png)

In this part, you will implement a piece of a relational query optimizer: Plan space search.

## Overview: Plan Space Search

You will now search the plan space of some cost estimates. For our database, this is similar to System R: the set of all left-deep trees, avoiding Cartesian products where possible. Unlike System R, we do not consider interesting orders, and further, we completely disallow Cartesian products in all queries. To search the plan space, we will utilize the dynamic programming algorithm used in the Selinger optimizer.

Before you begin, you should have a good idea of how the `QueryPlan` class is used \(see the [Skeleton Code](skeleton-code.md) section\) and how query operators fit together. For example, to implement a simple query with a single selection predicate:

```java
/**
 * SELECT * FROM myTableName WHERE stringAttr = 'CS 186'
 */
QueryOperator source = SequentialScanOperator(transaction, myTableName);
QueryOperator select = SelectOperator(source, 'stringAttr', PredicateOperator.EQUALS, "CS 186");

int estimatedIOCost = select.estimateIOCost(); // estimate I/O cost
Iterator<Record> iter = select.iterator(); // iterator over the results
```

A tree of `QueryOperator` objects is formed when we have multiple tables joined together. The current implementation of `QueryPlan#execute`, which is called by the user to run the query, is to join all tables in the order given by the user: if the user says `SELECT * FROM t1 JOIN t2 ON .. JOIN t3 ON ..`, then it scans `t1`, then joins `t2`, then joins `t3`. This will perform poorly in many cases, so your task is to implement the dynamic programming algorithm to join the tables together in a better order.

You will have to implement the `QueryPlan#execute` method. To do so, you will also have to implement two helper methods: `QueryPlan#minCostSingleAccess` \(pass 1 of the dynamic programming algorithm\) and `QueryPlan#minCostJoins` \(pass i &gt; 1\).

## Visualizing the Naive Query Optimizer
This section is optional, but we recommend that you run through the steps.

Our database supports an `EXPLAIN` command which outputs the query plan for a given query. Let's test out our current query optimizer! Navigate to `CommandLineInterface.java` and run the code to start our CLI. This should open a new panel in IntelliJ at the bottom. Click on this panel. We've provided 3 demo tables (Students, Courses, Enrollments). Let's try running the following query:

```sql
SELECT * FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid;
```

Let's display the query plan used to execute the above query by running the following command:

```sql
EXPLAIN SELECT * FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid;
```

An estimated 603 I/Os, a very costly query! Our current naive query optimizer joins the table in the order given and only uses SNLJs for joins, which can become very expensive. Let's try a more complex query. The following computes the distribution of majors in CS186.

```sql
SELECT c.name, s.major, COUNT(*) FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid INNER JOIN Courses AS c ON e.cid = c.cid WHERE c.name = 'CS 186' GROUP BY s.major, c.name;
```

Like before, let's inspect the query plan.

```sql
EXPLAIN SELECT c.name, s.major, COUNT(*) FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid INNER JOIN Courses AS c ON e.cid = c.cid WHERE c.name = 'CS 186' GROUP BY s.major, c.name;
```

This query also performs very poorly. Run `exit` to terminate the CLI. In the next few tasks, we'll implement an optimizer that will drastically improve the cost of our queries!

## Your Tasks

Note that you may **not** modify the signature of any methods or classes that we provide to you, but you're free to add helper methods. Also, you should only modify `query/QueryPlan.java` in this part.

### Task 5: Single Table Access Selection \(Pass 1\)

Recall that the first part of the search algorithm involves finding the lowest estimated cost plans for accessing each table individually \(pass i involves finding the best plans for sets of i tables, so pass 1 involves finding the best plans for sets of 1 table\).

This functionality should be implemented in the `QueryPlan#minCostSingleAccess` helper method, which takes a table and returns the optimal `QueryOperator` for scanning the table.

In our database, we only consider two types of table scans: a sequential, full table scan \(`SequentialScanOperator`\) and an index scan \(`IndexScanOperator`\), which requires an index and filtering predicate on a column.

You should first calculate the estimated I/O cost of a sequential scan, since this is always possible \(it's the default option: we only move away from it in favor of index scans if the index scan is both possible and more efficient\).

Then, if there are any indices on any column of the table that we have a selection predicate on, you should calculate the estimated I/O cost of doing an index scan on that column. If any of these are more efficient than the sequential scan, take the best one.

Finally, as part of a heuristic-based optimization covered in class, you should push down any selection predicates that involve solely the table \(see `QueryPlan#addEligibleSelections`\).

This should leave you with a query operator beginning with a sequential or index scan operator, followed by zero or more `SelectOperator`s.

After you have implemented `QueryPlan#minCostSingleAccess`, you should be passing all of the tests in `TestSingleAccess`. These tests do not involve any joins.

### **Task 6: Join Selection \(Pass i &gt; 1\)**

Recall that for i &gt; 1, pass i of the dynamic programming algorithm takes in optimal plans for joining together all possible sets of i - 1 tables \(except those involving cartesian products\), and returns optimal plans for joining together all possible sets of i tables \(again excluding those with cartesian products\).

We represent the state between two passes as a mapping from sets of strings \(table names\) to the corresponding optimal `QueryOperator`. You will need to implement the logic for pass i \(i &gt; 1\) of the search algorithm in the `QueryPlan#minCostJoins` helper method.

This method should, given a mapping from sets of i - 1 tables to the optimal plan for joining together those i - 1 tables, return a mapping from sets of i tables to the optimal left-deep plan for joining all sets of i tables \(except those with cartesian products\).

You should use the list of explicit join conditions added when the user calls the `QueryPlan#join` method to identify potential joins.

After implementing this method you should be passing `TestOptimizationJoins#testMinCostJoins`

**Note:** you should not add any selection predicates in this method. This is because in our database, we only allow two column predicates in the join condition, and a conjunction of single column predicates otherwise, so the only unprocessed selection predicates in pass i &gt; 1 are the join conditions. _This is not generally the case!_ SQL queries can contain selection predicates that can _not_ be processed until multiple tables have been joined together, for example:

```sql
SELECT * FROM t1, t2, t3, t4 WHERE (t1.a = t2.b OR t2.b = t2.c)
```

where the single predicate cannot be evaluated until after `t1`, `t2`, _and_ `t3` have been joined together. Therefore, a database that supports all of SQL would have to push down predicates after each pass of the search algorithm.

### Task 7: Optimal Plan Selection

Your final task is to write the outermost driver method of the optimizer, `QueryPlan#execute`, which should utilize the two helper methods you have implemented to find the best query plan.

You will need to add the remaining group by and projection operators that are a part of the query, but have not yet been added to the query plan \(see the private helper methods implemented for you in the `QueryPlan` class\).

**Note:** The tables in `QueryPlan` are kept in the variable `tableNames`. 

After this, you should pass all the tests we have provided to you in `database.query.*`.

## Visualizing the Query Optimizer
This section is also optional, but we recommend that you run through the steps.

Now that we've finished implementing a better query optimizer, let's visualize the results and compare it with the [naive query optimizer](https://cs186.gitbook.io/project/assignments/proj3/part-2-query-optimization#optional-visualizing-the-naive-query-optimizer)! Navigate to `CommandLineInterface.java` and run the code to start our CLI. Let's try running the following two queries again:

```sql
EXPLAIN SELECT * FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid;
```

```sql
EXPLAIN SELECT c.name, s.major, COUNT(*) FROM Students AS s INNER JOIN Enrollments AS e ON s.sid = e.sid INNER JOIN Courses AS c ON e.cid = c.cid WHERE c.name = 'CS 186' GROUP BY s.major, c.name;
```

The outputted query plans are much better than before! Notice how we now push down selects and use more efficient joins.

## Submission

Follow the submission instructions [here](submitting-the-assignment.md) for the Project 3 Part 2 assignment on Gradescope. If you completed everything you should be passing all the tests in the following files:

* `database.query.TestNestedLoopJoin`
* `database.query.TestGraceHashJoin`
* `database.query.TestSortOperator`
* `database.query.TestSingleAccess`
* `database.query.TestOptimizationJoins`
* `database.query.TestBasicQuery`

