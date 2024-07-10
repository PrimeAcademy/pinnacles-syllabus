# SQL Aggregate Functions

- Instructor Note:
  - Use the `"creatures"` database schema from the relational-SQL lectures.

---

You're a software developer now. Why not also be a data scientist? 🙂

(You won't be a data scientist after this lecture. But! You will have some very powerful tools that data scientists often make use of in some of the cool SQL stuff they do.)

## COUNT: Our First Aggregate Function!

Did you know you can count stuff with SQL? `COUNT` is an **Aggregate Function**. In SQL, an aggregate function performs a calculation on a set of values and returns a single value.

The most straightforward way to use the `COUNT` function is to just count *how many rows are in a query result*:

```SQL
-- Count all the creatures:
SELECT COUNT(*) FROM "creatures";

-- Count all the creatures who are into Photography:
SELECT COUNT(*) FROM "creatures"
  JOIN "creatures_hobbies"
    ON "creatures"."id" = "creatures_hobbies"."creature_id"
  JOIN "hobbies"
    ON "creatures_hobbies"."hobby_id" = "hobbies"."id"
  WHERE "hobbies"."name" = 'Photography';
```

---

## GROUP BY: Unlocking the Real Power of Aggregate Functions!

### How Many Things Does Each Creature Have?

When counting, sometimes we need to do more than just count the rows that are returned by a single query. Say we want the names of the creatures **and** how many things each creature has. If our `"creatures"` and `"things"` tables still look like this:

| id  | name     |
| --- | -----    |
| 1   | Sonic    | -- 👈 Has 1 thing.
| 2   | Alf      | -- 👈 Has 5 things.
| 3   | Moomin   | -- 👈 Has 0 things.

| id  | name   | value  | creature_id |
| --- | ------ | ------ | ----------- |
| 1   | Rock   | 0.99   | 1           |
| 2   | Violin | 350.99 | 2           |
| 3   | Sock   | 1.00   | 2           |
| 4   | Frog   | 5.99   | 2           |
| 5   | Cat    | 99.99  | 2           |
| 6   | Hat    | 25.00  | 2           |

We need to write a SQL query that'll give us this result:

| name    | thing_count |
|------   |------------ |
| Sonic   |  1          |
| Moomin  |  0          |
| Alf     |  5          |

### Let's Talk about `GROUP BY`

In order to do this, we need to tell SQL how to *group* our data together, then count how many rows are in each group. We need to learn a new trick: `GROUP BY`! 🎉

The `GROUP BY` clause sorts rows that **have the same values** into bundles. After bundling these rows together, we can then run **aggregate functions** (like `COUNT`!) on each group.

Here's a basic example:

Let's break down the general process using `GROUP BY` with aggregate functions:

1. `SELECT` Clause: We choose the columns that we need to obtain our result. 
    * **IMPORTANT IDEA**: Every column in the `SELECT` clause that is not used in an aggregate function must be included in the `GROUP BY` clause.
2. `FROM` Clause: We specify the table that we'll be getting data out of.

3. `JOIN` Clause (*if needed*): We can join other tables.

4. `GROUP BY` Clause: We specify which column(s) should dictate how we group the data.

5. Aggregate Functions: We apply aggregate functions to the grouped data to get summarized results.

### Let's Do It

```sql
SELECT
  "creatures"."name",
  COUNT("things"."creature_id") AS "thing_count"
FROM "creatures"
  LEFT JOIN "things"
    ON "creatures"."id" = "things"."creature_id"
  GROUP BY "creatures"."name";
```

1. `FROM "creatures"`: We need the `"name"` data from the creatures table. (It's what we're going to be grouping on!)

2. `LEFT JOIN "things" ON "creatures"."id" = "things"."creature_id"`: We're using `LEFT JOIN` so we don't lose Moomin! (Moomin doesn't have any things.)

3. `SELECT "creatures"."name", COUNT("things"."id")`: 
    * Select the name of the creature. (For grouping purposes!)
    * Count the number of ids in the things table for each creature.

4. `GROUP BY "creatures"."name"`: Group the results by the `"name"` of the creature. This means that all rows with the same creature name will be grouped together.
    * In this example, we have three creatures, so we'll end up with three groups.

### And We Can Do Much More Than Count!

Here are the five aggregate functions that exist in every flavor of SQL:

* COUNT(`"some_column"`): Counts the number of non-NULL values within the column.
* SUM(`"some_column"`): Adds up the values in the column.
* AVG(`"some_column"`): Calculates the average value of the column.
* MIN(`"some_column"`): Finds the minimum value of the column.
* MAX(`"some_column"`): Finds the maximum value of the column.

And! The different flavors of SQL (PostgreSQL, MySQL, Microsoft SQL Server...) have their own additional aggregate functions built in! Here are the PostgreSQL ones:
* https://www.postgresql.org/docs/16/functions-aggregate.html
    * There's quite a bit of powerful stuff in there...


Here's an example of how we could use `SUM` to calculate the **total value** of each creature's things:

```sql
-- Spoiler Alert:
-- Structurally, this is basically the same query as the previous
-- COUNT example. :)
SELECT
  "creatures"."name",
  SUM("things"."value") AS "total_value_of_things"
FROM "creatures"
  LEFT JOIN "things"
    ON "creatures"."id" = "things"."creature_id"
  GROUP BY "creatures"."name";
```

Note that Moomin's `"total_value_of_things"` is `NULL` in this output. 


Anywho, time to practice using `GROUP BY` and aggregate functions!


## Additional Resources

- 🔥[PostgreSQL's Aggregate Functions Tutorial](https://www.postgresql.org/docs/current/tutorial-agg.html)
- [SQL Group By](https://www.w3schools.com/sql/sql_groupby.asp)
- [SQL Min and Max](https://www.w3schools.com/sql/sql_min_max.asp)
- [SQL Count, Avg, Sum](https://www.w3schools.com/sql/sql_count_avg_sum.asp)