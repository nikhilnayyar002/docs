https://www.postgresql.org/docs

PostgreSQL uses a client/server model.

A server process, which manages the database files, accepts connections to the database from client applications, and performs database actions on behalf of the clients. The database server program is called `postgres`.

The PostgreSQL server can handle multiple concurrent connections from clients. To achieve this it starts (“forks”) a new process for each connection. From that point on, the client and the new server process communicate without intervention by the original postgres process. Thus, the supervisor server process is always running, waiting for client connections, whereas client and associated server processes come and go

A running PostgreSQL server can manage many databases. Typically, a separate database is used for each project or for each user.

```
createdb mydb
```

Once you have created a database, you can access it by `psql`

```
$ psql mydb
```
If you do not supply the database name then it will default to your user account name.  

```
mydb=> SELECT version();
```

The `psql` program has a number of internal commands that are not SQL commands. They begin with the backslash character, `“\”`

```
mydb=> \h // help
mydb=> \q // quit
```

```
CREATE TABLE weather (
    city            varchar(80),
    temp_hi         int,           -- high temperature
    prcp            real,          -- precipitation
    date            date
);
```
You can enter this into psql with the line breaks. `psql` will recognize that the command is not terminated until the semicolon. Two dashes (“`--`”) introduce comments. SQL is case-insensitive about key words and identifiers, except when identifiers are double-quoted to preserve the case.

```
DROP TABLE tablename;

INSERT INTO weather VALUES ('San Francisco', 46, 50, 0.25, '1994-11-27');
INSERT INTO cities VALUES ('San Francisco', '(-194.0, 53.0)');

INSERT INTO weather (city, temp_hi, date)
    VALUES ('San Francisco', 57, '1994-11-29');
```
You can list the columns in a different order if you wish or even omit some columns, e.g., if the precipitation is unknown

```
SELECT * FROM weather; 
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
SELECT * FROM weather
    WHERE city = 'San Francisco' AND prcp > 0.0;
SELECT * FROM weather
    ORDER BY city, temp_lo;
SELECT DISTINCT city FROM weather;
```

Queries that access multiple tables (or multiple instances of the same table) at one time are called join queries
```
SELECT * FROM weather JOIN cities ON city = name;
SELECT city, temp_lo, temp_hi, prcp, date, location -- name column is same as city ommited
    FROM weather JOIN cities ON city = name;
```

If there were duplicate column names in the two tables you'd need to qualify the column names
```
SELECT weather.city, cities.location
    FROM weather JOIN cities ON weather.city = cities.name;
```

If no matching row is found we want some “empty values” to be substituted for the cities table's columns. This kind of query is called an outer join. (The joins we have seen so far are inner joins.)
```
SELECT *
    FROM weather LEFT OUTER JOIN cities ON weather.city = cities.name;

SELECT *
    FROM weather w JOIN cities c ON w.city = c.name;
```

An aggregate function computes a single result from multiple input rows. For example, there are aggregates to compute the `count`, `sum`, `avg` (average), `max` (maximum) and `min` (minimum) over a set of rows
```
SELECT max(temp_lo) FROM weather;
```

If we wanted to know what city (or cities) that reading occurred in
```
SELECT city FROM weather
    WHERE temp_lo = (SELECT max(temp_lo) FROM weather); --  the subquery
```

Aggregates are also very useful in combination with `GROUP BY` clauses. For example, we can get the number of readings and the maximum low temperature observed in each city with:
```
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city;

     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     2 |  46
```

We can filter these grouped rows using HAVING:
```
SELECT city, count(*), max(temp_lo)
    FROM weather
    GROUP BY city
    HAVING max(temp_lo) < 40;
```

if we only care about cities whose names begin with “S”,
```
SELECT city, count(*), max(temp_lo)
    FROM weather
    WHERE city LIKE 'S%'
    GROUP BY city;
```

`WHERE` selects input rows before groups and aggregates are computed. `HAVING` selects group rows after groups and aggregates are computed.

`FILTER` is a per-aggregate option:
```
SELECT city, count(*) FILTER (WHERE temp_lo < 45), max(temp_lo)
    FROM weather
    GROUP BY city;

     city      | count | max
---------------+-------+-----
 Hayward       |     1 |  37
 San Francisco |     1 |  46
```
`FILTER` is much like `WHERE`, except that it removes rows only from the input of the particular aggregate function that it is attached to. Here, the `count` aggregate counts only rows with `temp_lo` below 45; but the `max` aggregate is still applied to all rows, so it still finds the reading of 46.

Suppose you discover the temperature readings are all off by 2 degrees after November 28. You can correct the data as follows:
```
UPDATE weather
    SET temp_hi = temp_hi - 2,  temp_lo = temp_lo - 2
    WHERE date > '1994-11-28';
```
```
DELETE FROM weather WHERE city = 'Hayward';
```

Suppose the combined listing of weather records and city location is of particular interest to your application, but you do not want to type the query each time you need it. You can create a view over the query, which gives a name to the query that you can refer to like an ordinary table:
```
CREATE VIEW myview AS
    SELECT name, temp_lo, temp_hi, prcp, date, location
        FROM weather, cities
        WHERE city = name;

SELECT * FROM myview;
```

You want to make sure that no one can insert rows in the weather table that do not have a matching entry in the cities table. This is called maintaining the referential integrity of your data
```
CREATE TABLE cities (
        name     varchar(80) primary key,
        location point
);
CREATE TABLE weather (
        city      varchar(80) references cities(name),
        temp_lo   int,
        prcp      real,
        date      date
);
```

The essential point of a transaction is that it bundles multiple steps into a single, all-or-nothing operation. In PostgreSQL, a transaction is set up by surrounding the SQL commands of the transaction with BEGIN and COMMIT commands.
```
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
-- etc etc
COMMIT;
```

If, partway through the transaction, we decide we do not want to commit (perhaps we just noticed that Alice's balance went negative), we can issue the command `ROLLBACK` instead of `COMMIT`, and all our updates so far will be canceled.

Savepoints allow you to selectively discard parts of the transaction, while committing the rest. After defining a savepoint with `SAVEPOINT`, you can if needed roll back to the savepoint with `ROLLBACK TO`.

After rolling back to a savepoint, it continues to be defined, so you can roll back to it several times. Conversely, if you are sure you won't need to roll back to a particular savepoint again, it can be released, so the system can free some resources. Keep in mind that either releasing or rolling back to a savepoint will automatically release all savepoints that were defined after it.

suppose we debit $100.00 from Alice's account, and credit Bob's account, only to find later that we should have credited Wally's account. We could do it using savepoints like this:
```
BEGIN;
UPDATE accounts SET balance = balance - 100.00
    WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Bob';
-- oops ... forget that and use Wally's account
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00
    WHERE name = 'Wally';
COMMIT;
```

`avg(salary)` is an aggregate function (it wants to "squish" all rows into a single value). If you want to see the average salary for each department, you must use GROUP BY.
```
SELECT depname, avg(salary) 
FROM empsalary
GROUP BY depname;
```

If you want to see every single employee and have the average salary listed next to them on every row, you use a Window Function by adding the OVER clause.
```
SELECT depname, empno, salary, avg(salary) OVER () 
FROM empsalary;
```

If you were trying to see how each employee's salary compares to their department's average specifically, you would use avg(salary) OVER (PARTITION BY depname). This would give you every row, but the average would change based on which department that employee belongs to.
```
SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname)
FROM empsalary;

  depname  | empno | salary |          avg
-----------+-------+--------+-----------------------
 develop   |    11 |   5200 | 5020.0000000000000000
 develop   |     7 |   4200 | 5020.0000000000000000
 personnel |     5 |   3500 | 3700.0000000000000000
 personnel |     2 |   3900 | 3700.0000000000000000
```

`OVER` is the "magic key" that turns a standard function into a Window Function.
You can also control the order in which rows are processed by window functions using `ORDER BY` within `OVER`
```
SELECT depname, empno, salary,
       row_number() OVER (PARTITION BY depname ORDER BY salary DESC)
FROM empsalary;

  depname  | empno | salary | row_number
-----------+-------+--------+------------
 develop   |     8 |   6000 |          1
 develop   |    10 |   5200 |          2
 personnel |     2 |   3900 |          1
 personnel |     5 |   3500 |          2
```

a row removed because it does not meet the `WHERE` condition is not seen by any window function. A query can contain multiple window functions.

```
SELECT depname, empno, salary, enroll_date
FROM
  (SELECT depname, empno, salary, enroll_date,
     row_number() OVER (PARTITION BY depname ORDER BY salary DESC, empno) AS pos
     FROM empsalary
  ) AS ss
WHERE pos < 3;

SELECT sum(salary) OVER w, avg(salary) OVER w
  FROM empsalary
  WINDOW w AS (PARTITION BY depname ORDER BY salary DESC);
```

capitals are also cities
```
CREATE TABLE cities (
  name       text,
  population real,
  elevation  int     -- (in ft)
);

CREATE TABLE capitals (
  state      char(2) UNIQUE NOT NULL
) INHERITS (cities);
```
When you tell a table to `INHERIT` from another, you are creating a parent-child relationship where the parent table acts as a "view" of all its children. when you query the parent table (`cities`), the database assumes you want to see every row that qualifies as a "city." Since every capital is a city (by virtue of inheritance), Postgres automatically scans the `capitals` table and includes those rows in your result set. To find all the cities that are not state capitals use `FROM ONLY cities` 

