# Grouping and Sorting Data

## Objectives

- Explain the importance of grouping and sorting data stored in a database
- Group and sort data with the `GROUP BY()` and `ORDER BY()` keywords
- Craft advanced queries using aggregator functions along with sorting keywords
  and other conditional clauses

## Grouping and Sorting Data

SQL isn't picky about how it returns data to you, based on your queries. It will
simply return the relevant table rows in the order in which they exist in the
table. This is often insufficient for the purposes of data analysis and
organization.

How common is it to order a list of items alphabetically? Or numerically from
least to greatest?

We can tell our SQL queries and aggregate functions to group and sort our data
using several clauses:

- `ORDER BY()`
- `LIMIT`
- `GROUP BY()`
- `HAVING` and `WHERE`
- `ASC`/`DESC`

Let's take a closer look at how we use these keywords to narrow our search
criteria as well as to order and group it.

## Setting up the Database

Some cats are very famous, and accordingly very wealthy. Our Pets Database will
have a `cats` table in which each cat has a name, age, breed, and net worth. Our
database will also have an `owners` table and `cats_owners` join table so that a
cat can have many owners and an owner can have many cats.

**Creating the Database:**

Create the database in your terminal with the following:

```bash
sqlite3 pets_database.db
```

**Creating the tables:**

In the `sqlite3>` prompt in your terminal:

**`cats` table:**

```sql
CREATE TABLE cats (
id INTEGER PRIMARY KEY,
name TEXT,
age INTEGER,
breed TEXT,
net_worth INTEGER
);
```

**`owners` Table:**

```sql
CREATE TABLE owners (id INTEGER PRIMARY KEY, name TEXT);
```

**`cats_owners` Table:**

```sql
CREATE TABLE cats_owners (
cat_id INTEGER,
owner_id INTEGER
);
```

**Inserting the values:**

**`cats`:**

```sql
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (1, "Maru", 3, "Scottish Fold", 1000000);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (2, "Hana", 1, "Tabby", 21800);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (3, "Grumpy Cat", 4, "Persian", 181600);
INSERT INTO cats (id, name, age, breed, net_worth) VALUES (4, "Lil\' Bub", 2, "Tortoiseshell", 2000000);
```

**`owners`:**

```sql
INSERT INTO owners (name) VALUES ("mugumogu");
INSERT INTO owners (name) VALUES ("Sophie");
INSERT INTO owners (name) VALUES ("Penny");
```

**`cats_owners`:**

```sql
INSERT INTO cats_owners (cat_id, owner_id) VALUES (2, 2);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (3, 3);
INSERT INTO cats_owners (cat_id, owner_id) VALUES (1, 2);
```

### Code Along I: `ORDER BY()`

#### Syntax

```sql
SELECT column_name, column_name
FROM table_name
ORDER BY column_name DESC, column_name ASC|DESC;
```

`ORDER BY()` will automatically sort the returned values in ascending order. Use
the `DESC` keyword, as above, to sort in descending order.

#### Exercise

Imagine you're working for an important investment firm in Manhattan. The
investors are interested in investing in a lucrative and popular cat. They need
your help to decide which cat that will be. They want a list of famous and
wealthy cats. We can do that with a basic `SELECT` statement:

```sql
SELECT * FROM cats WHERE net_worth > 0;
```

This will return:

```bash
name             age         breed          net_worth
---------------  ----------  -------------  ----------
Maru             3           Scottish Fold  1000000
Hana             1           Tabby          21800
Grumpy Cat       4           Persian        181600
Lil\' Bub         2           Tortoiseshell  2000000
```

Our investors are busy people though. They don't have time to manually sort
through this list of cats for the best candidate. They want you to return the
list to them with the cats sorted by net worth, from greatest to least.  

We can do so with the following lines:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC;
```

This will return:

```bash
name             age         breed          net_worth
---------------  ----------  -------------  ----------
Lil\' Bub        2           Tortoiseshell  2000000
Maru             3           Scottish Fold  1000000
Grumpy Cat       4           Persian        181600
Hana             1           Tabby          21800
```

### Code Along II: The `LIMIT` Keyword

Turns out our investors are very impatient. They don't want to review the list
themselves, they just want you to return to them the wealthiest cat. We can
accomplish this by using the `LIMIT` keyword with the above query:

```sql
SELECT * FROM cats ORDER BY(net_worth) DESC LIMIT 1;
```

Which will return:

```text
name             age         breed          net_worth
---------------  ----------  -------------  ----------
Lil\' Bub        2           Tortoiseshell  2000000
```

The `LIMIT` keyword specifies how many of the records resulting from the query
you'd like to actually return.

### Code Along III: `GROUP BY()`

The `GROUP BY()` keyword is very similar to `ORDER BY()`. The only difference is that [`ORDER BY()` sorts the resulting data set of basic queries while `GROUP BY()` sorts the result sets of aggregate functions](http://www.thetricky.net/mySQL/GROUP+BY+vs+ORDER+BY).

#### Syntax

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name;
```

#### Exercise

Let's calculate the sum of the net worth of all of the cats, grouped by owner name:

```sql
SELECT owner.name, SUM(cats.net_worth)
FROM owners
INNER JOIN cats_owners
ON owners.id = cats_owners.owner_id
JOIN cats ON cats_owners.cat_id = cats.id
GROUP BY owners.name;
```

This should return:

```text
owners.name      SUM(cats.networth)
---------------  ----------
Penny            181600
Sophie           1021800
```

In the above query, we've implemented _two_ joins. First, we're joining `owners`
and `cat_owners` on `owners.id = cats_owners.owner_id`. This first joined table
would look like the following if we were to query it:

```text
owners.id  owners.name      cat_owners.cat_id  cat_owners.owner_id
---------  -----------      -----------------  -----------------
2          Sophie           2                  2
3          Penny            3                  3
2          Sophie           1                  2
```

With this table, we then implement a _second_ join with `cats` on
`cats_owners.cat_id = cats.id`. To better understand this, try running the
provided query, but select _everything_ rather than just the owner's name and
the sum of their cats' net worth, and remove the `GROUP BY` line. You'll be able
to see all three tables have been joined.

In our example query above, when we use the `SUM(cats.net_worth)` aggregator in
conjunction with `GROUP BY`, the combination changes the way that our query
behaves. Without `GROUP BY`, we would get a sum of the net worth of all the
cats:

```text
owners.name      SUM(cats.networth)
---------------  ----------
Sophie           1203400
```

By adding `GROUP BY`, we now get the net_worth of all cats _by owner_. In our
original data, Sophie is the owner of Maru and Hana (100000 + 21800), while
Penny is the owner of Lil' Bub (181600).

`SUM` looks at all of the values in the `net_worth` column of the `cats`
table (or whichever column you specify in parentheses) and takes the sum of the
those values, but only _after those cats have been grouped_.

> **Note**: If you forget to add `SUM` here and just try to get
> `cats.net_worth`, you'll still group by owner, but it will only display the
> first cat's net worth, not the aggregate.

### Code Along IV: `HAVING` vs `WHERE` clause

Suppose we have a table called `employee_bonus` as shown below. Note that the
table has multiple entries for employees Abigail and Matthew.

**`employee_bonus`**:

Employee   | Bonus
-----------|-------
Matthew    |1000|
Abigail    |2000|
Matthew    |500|
Tom         |700|
Abigail     |1250|  

To calculate the total bonus that each employee received, we would write a SQL
statement like this:  

```sql
SELECT employee, SUM(bonus) from employee_bonus group by employee;
```  

This should return:  

Employee   | Bonus
-----------|-------
Abigail    |3250|
Matthew    |1500|
Tom        |700|

Now, suppose we wanted to find the employees who received more than $1,000 in
bonuses for the year of 2007. You might think that we could write a query like
this:  

```sql  
BAD SQL:
SELECT employee, SUM(bonus) FROM employee_bonus
GROUP BY employee WHERE SUM(bonus) > 1000;
```  

Unfortunately, the above will not work because the `WHERE` clause doesn’t work
with aggregates – like `SUM`, `AVG`, `MAX`, etc. What we need to use is the
`HAVING` clause. The `HAVING` clause was added to SQL so that we could compare
aggregates to other values – just how the `WHERE` clause can be used with
non-aggregates. Now, the correct SQL will look like this:

```sql
GOOD SQL:
SELECT employee, SUM(bonus) FROM employee_bonus
GROUP BY employee HAVING SUM(bonus) > 1000;
```  

#### Difference between `HAVING` and `WHERE` clause

The difference between the `HAVING` and `WHERE` clause in SQL is that the
`WHERE` clause can not be used with aggregates but the `HAVING` clause can.
HAVING filters out groups of rows, created by 'GROUP BY' and WHERE filters out
rows. Another way to think of it is that the `HAVING` clause is an additional
filter to the `WHERE` clause.

- HAVING supports aggregate functions as it has to work with groups of rows. so
  for example, if there are multiple integers in a group it can filter out the
  groups with a low average, a high total (sum) or count how many rows are in
  the group.

- WHERE on the other hand deals with each row individually, so aggregate
  functions wouldn't work for what would you be aggregating.

Also, note syntax differences:

HAVING is __after__ GROUP BY and WHERE is __before__ GROUP BY changing the order
will produce a  syntax error.

```sql
SELECT
FROM
JOIN
  ON
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

## Resources

- [`HAVING` vs `WHERE` clauses](https://www.essentialsql.com/what-is-the-difference-between-where-and-having-clauses/)

- [Video Review- SQL Joins Overview](https://www.youtube.com/watch?v=qfB1MRnzk4g) 

<p class='util--hide'>View <a href='https://learn.co/lessons/sql-grouping-and-sorting-readme'>Grouping and Sorting Data</a> on Learn.co and start learning to code for free.</p>
