https://www.postgresql.org/docs/current/sql.html

https://www.postgresql.org/docs/current/datatype.html

A string constant in SQL is an arbitrary sequence of characters bounded by single quotes.
To include a single-quote character within a string constant, write two adjacent single quotes, e.g., `'Dianne''s horse'`.
PostgreSQL provides another way, called “dollar quoting”, to write string constants.
`$$Dianne's horse$$`. inside the dollar-quoted string, single quotes can be used without needing to be escaped. Indeed, no characters inside a dollar-quoted string are ever escaped

A numeric constant that contains neither a decimal point nor an exponent is initially presumed to be type `integer` if its value fits in type `integer` (32 bits); otherwise it is presumed to be type `bigint` if its value fits in type `bigint` (64 bits); otherwise it is taken to be type `numeric`. Constants that contain decimal points and/or exponents are always initially presumed to be type `numeric`.

you can force a numeric value to be treated as type real (float4) by writing
`REAL '1.23'`

```
SELECT ARRAY[[1,2],[3,4]]; --  {{1,2},{3,4}}
SELECT ROW(1,2.5,'this is a test');
```

The order of evaluation of subexpressions is not defined. When it is essential to force evaluation order, a `CASE` construct can be used. this is an untrustworthy way of trying to avoid division by zero in a WHERE clause:

```
SELECT ... WHERE x > 0 AND y/x > 1.5; // unsafe
SELECT ... WHERE CASE WHEN x > 0 THEN y/x > 1.5 ELSE false END; // safe
```

One limitation of the technique illustrated above is that it does not prevent early evaluation of constant subexpressions (`1/0` instead of `y/x`). 

```
CREATE FUNCTION concat_lower_or_upper(a text, b text, uppercase boolean DEFAULT false)
RETURNS text
AS
$$
 SELECT CASE
        WHEN $3 THEN UPPER($1 || ' ' || $2)
        ELSE LOWER($1 || ' ' || $2)
        END;
$$
LANGUAGE SQL IMMUTABLE STRICT;

SELECT concat_lower_or_upper('Hello', 'World', true); -- HELLO WORLD
```

If no default value is declared explicitly, the default value is the null value.
```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric DEFAULT 9.99
);
```
The default value can be an expression, which will be evaluated whenever the default value is inserted (not when the table is created).

An identity column is a special column that is generated automatically from an implicit sequence
```
CREATE TABLE people (
    id bigint GENERATED ALWAYS AS IDENTITY,
    ...,
);

INSERT INTO people (name, address) VALUES ('A', 'foo');
-- 1 | A    | foo
```

A generated column is a special column that is always computed from other columns. 
A stored generated column is computed when it is written. A virtual generated column occupies no storage and is computed when it is read.
```
CREATE TABLE people (
    ...,
    height_cm numeric,
    height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```

A check constraint is the most generic constraint type.
```
CREATE TABLE products (
    price numeric CHECK (price > 0)
);
```
A check constraint can also refer to several columns
```
CREATE TABLE products (
    price numeric CHECK (price > 0),
    discounted_price numeric CHECK (discounted_price > 0),
    CHECK (price > discounted_price)
);
```
```
    product_no integer NOT NULL,
    price numeric NOT NULL CHECK (price > 0)
    product_no integer UNIQUE,

    a integer,
    c integer,
    UNIQUE (a, c) -- combination of values is unique across the whole table
```

by default, NULL is never equal to another NULL. it is possible to store duplicate rows that contain a null. This behavior can be changed by adding the clause `NULLS NOT DISTINCT`
```
product_no integer UNIQUE NULLS NOT DISTINCT,
```

A primary key constraint indicates that a column, or group of columns, can be used as a unique identifier for rows in the table. This requires that the values be both unique and not null. 
```
    product_no integer UNIQUE NOT NULL,
    product_no integer PRIMARY KEY,
```
A table can have at most one primary key. 

A foreign key constraint specifies that the values in a column (or a group of columns) must match the values appearing in some row of another table.
```
product_no integer REFERENCES products (product_no),

--- in absence of a column list the primary key of the referenced table is used 
product_no integer REFERENCES products,

  ...
  FOREIGN KEY (b, c) REFERENCES other_table (c1, c2)
```
```
CREATE TABLE products (
    product_no integer PRIMARY KEY,
);

CREATE TABLE orders (
    order_id integer PRIMARY KEY,
    shipping_address text,
);

CREATE TABLE order_items (
    product_no integer REFERENCES products,
    order_id integer REFERENCES orders,
    quantity integer,
    PRIMARY KEY (product_no, order_id)
);
```
when someone wants to remove a product that is still referenced by an order (via order_items), we disallow it. If someone removes an order, the order items are removed as well:
```
    product_no integer REFERENCES products ON DELETE RESTRICT,
    order_id integer REFERENCES orders ON DELETE CASCADE,       
```

PostgreSQL provides a family of commands to make modifications to existing tables definition.
```
ALTER TABLE products ADD COLUMN description text CHECK (description <> '');
ALTER TABLE products DROP COLUMN description;
ALTER TABLE products ADD CHECK (name <> '');
ALTER TABLE products ADD CONSTRAINT some_name UNIQUE (product_no);
ALTER TABLE products ADD FOREIGN KEY (product_group_id) REFERENCES product_groups;
ALTER TABLE products ALTER COLUMN product_no SET NOT NULL;

ALTER TABLE products DROP CONSTRAINT some_name;
ALTER TABLE products ALTER COLUMN product_no DROP NOT NULL;

ALTER TABLE products ALTER COLUMN price SET DEFAULT 7.77;
ALTER TABLE products ALTER COLUMN price DROP DEFAULT;
ALTER TABLE products ALTER COLUMN price TYPE numeric(10,2);

ALTER TABLE products RENAME COLUMN product_no TO product_number;
ALTER TABLE products RENAME TO items;
```

A database contains one or more named schemas, which in turn contain tables. Schemas also contain other kinds of named objects, including data types, functions, and operators. Furthermore, tables, sequences, indexes, views, materialized views, and foreign tables share the same namespace, so that, for example, an index and a table must have different names if they are in the same schema. By default tables (and other objects) are automatically put into a schema named “public”.
```
CREATE SCHEMA myschema;

CREATE TABLE myschema.mytable (
 ...
);
DROP SCHEMA myschema CASCADE;
```
Often you will want to create a schema owned by someone else (since this is one of the ways to restrict the activities of your users to well-defined namespaces). The syntax for that is:
```
CREATE SCHEMA schema_name AUTHORIZATION user_name;
```

tables are often referred to by unqualified names, which consist of just the table name. The system determines which table is meant by following a search path, which is a list of schemas to look in. The first matching table in the search path is taken to be the one wanted

The first schema named in the search path is called the current schema. Aside from being the first schema searched, it is also the schema in which new tables will be created if the `CREATE TABLE` command does not specify a schema name

```
SHOW search_path; --  "$user", public
```
The first element specifies that a schema with the same name as the current user is to be searched. If no such schema exists, the entry is ignored. The second element refers to the public schema that we have seen already

```
SET search_path TO myschema,public;
```

By default, users cannot access any objects in schemas they do not own. To allow that, the owner of the schema must grant the `USAGE` privilege on the schema. 

Partitioning refers to splitting what is logically one large table into smaller physical pieces. The partitioned table itself is a “virtual” table having no storage of its own. Instead, the storage belongs to partitions.

All rows inserted into a partitioned table will be routed to the appropriate one of the partitions based on the values of the partition key column(s)

It is not possible to turn a regular table into a partitioned table or vice versa. However, it is possible to add an existing regular or partitioned table as a partition of a partitioned table, or remove a partition from a partitioned table turning it into a standalone table; this can simplify and speed up many maintenance processes.

```
CREATE TABLE measurement (
    city_id         int not null,
    logdate         date not null,
    peaktemp        int,
    unitsales       int
) PARTITION BY RANGE (logdate);

CREATE TABLE measurement_y2006m02 PARTITION OF measurement
    FOR VALUES FROM ('2006-02-01') TO ('2006-03-01');
CREATE TABLE measurement_y2006m03 PARTITION OF measurement
    FOR VALUES FROM ('2006-03-01') TO ('2006-04-01')
    TABLESPACE fasttablespace;
```

```
INSERT INTO products (product_no, name, price) VALUES (1, 'Cheese', DEFAULT);
INSERT INTO products DEFAULT VALUES;
INSERT INTO products (product_no, name, price) VALUES
    (1, 'Cheese', 9.99),
    (2, 'Bread', 1.99);
INSERT INTO products (product_no, name, price)
  SELECT product_no, name, price FROM new_products
    WHERE release_date = 'today';

UPDATE products SET price = price * 1.10;
UPDATE mytable SET a = 5, b = 3, c = 1 WHERE a > 0;

DELETE FROM products WHERE price = 10;
```
when using a serial column to provide unique identifiers, `RETURNING` can return the ID assigned to a new row
```
CREATE TABLE users (firstname text, lastname text, id serial primary key);
INSERT INTO users (firstname, lastname) VALUES ('Joe', 'Cool') RETURNING id;

UPDATE products SET price = price * 1.10
  WHERE price <= 99.99
  RETURNING name, old.price AS old_price, new.price AS new_price,
            new.price - old.price AS price_change;
```

`FROM T1 CROSS JOIN T2` = `FROM T1 INNER JOIN T2 ON TRUE`

The words `INNER` and `OUTER` are optional in all forms. `INNER` is the default; `LEFT`, `RIGHT`, and `FULL` imply an outer join.

joining `T1` and `T2` with `USING (a, b)` produces the join condition ON `T1.a = T2.a AND T1.b = T2.b`.

```
SELECT * FROM t1 CROSS JOIN t2;
SELECT * FROM t1 JOIN t2 ON t1.num = t2.num;
SELECT * FROM t1 JOIN t2 USING (num);
SELECT * FROM t1 LEFT JOIN t2 USING (num);

SELECT * FROM some_very_long_table_name s JOIN another_fairly_long_name a ON s.id = a.num;

SELECT * FROM (SELECT * FROM table1) AS alias_name;
SELECT *FROM (VALUES ('anne', 'smith'), ('bob', 'jones'), ('joe', 'blow'))
     AS names(first, last);

SELECT ... FROM fdt WHERE c1 > 5
SELECT ... FROM fdt WHERE c1 IN (1, 2, 3)
SELECT ... FROM fdt WHERE c1 IN (SELECT c1 FROM t2)
SELECT ... FROM fdt WHERE c1 IN (SELECT c3 FROM t2 WHERE c2 = fdt.c1 + 10)
```
In general, if a table is grouped, columns that are not listed in `GROUP BY`cannot be referenced except in aggregate expressions
```
SELECT x, sum(y) FROM test1 GROUP BY x;
```

When working with multiple tables, it can also be useful to ask for all the columns of a particular table:
```
SELECT tbl1.*, tbl2.a FROM ...

SELECT a AS value, b + c AS sum FROM ...
```

`UNION` effectively appends the result of `query2` to the result of `query1`. `INTERSECT` returns all rows that are both in the result of `query1` and in the result of `query2`. `EXCEPT` returns all rows that are in the result of `query1` but not in the result of `query2`. 

```
(SELECT a FROM b UNION SELECT x FROM y) LIMIT 10
```
```
SELECT a, b FROM table1 ORDER BY a ASC, c DESC;
SELECT a + b AS sum, c FROM table1 ORDER BY sum;
```
`LIMIT` and `OFFSET` allow you to retrieve just a portion of the rows that are generated by the rest of the query

```
SELECT * FROM (VALUES (1, 'one'), (2, 'two'), (3, 'three')) AS t (num,letter);
```
`WITH` provides a way to write auxiliary statements for use in a larger query.
```
WITH regional_sales AS (
    SELECT region, SUM(amount) AS total_sales
    FROM orders
    GROUP BY region
), top_regions AS (
    SELECT region
    FROM regional_sales
    WHERE total_sales > (SELECT SUM(total_sales)/10 FROM regional_sales)
)
SELECT region,
       product,
       SUM(quantity) AS product_units,
       SUM(amount) AS product_sales
FROM orders
WHERE region IN (SELECT region FROM top_regions)
GROUP BY region, product;

WITH moved_rows AS (
    DELETE FROM products
    WHERE
        "date" >= '2010-10-01' AND
        "date" < '2010-11-01'
    RETURNING *
)
INSERT INTO products_log
SELECT * FROM moved_rows;
```

https://www.postgresql.org/docs/current/functions-matching.html

https://www.postgresql.org/docs/current/functions-formatting.html

https://www.postgresql.org/docs/current/functions-datetime.html

```
EXISTS (subquery)
-- The argument of EXISTS is an arbitrary SELECT statement, or subquery. The subquery is evaluated to determine whether it returns any rows.

expression IN (subquery)
-- The right-hand side is a parenthesized subquery, which must return exactly one column. 

expression NOT IN (subquery)
row_constructor IN (subquery)
expression operator ANY (subquery)
expression operator SOME (subquery)
```
```
SELECT text 'Origin' AS "label", point '(0,0)' AS "value"; -- 1 row, 2 col
SELECT |/ 40 AS "square root of 40"; -- 1 row, 1 col
SELECT text 'abc' || 'def' AS "text and unknown"; -- abcdef
SELECT @ '-4.5' AS "abs"; -- float8 assumed
SELECT ~ CAST('20' AS int8) AS "negation";
```
```
CREATE TABLE test1 (
    id integer,
    content varchar
);
CREATE INDEX test1_id_index ON test1 (id);
DROP INDEX test1_id_index;
```
Indexes can also benefit `UPDATE` and `DELETE` commands with search conditions. Indexes can moreover be used in join searches. Thus, an index defined on a column that is part of a join condition can also significantly speed up queries with joins.

A multicolumn B-tree index can be used with query conditions that involve any subset of the index's columns, but the index is most efficient when there are constraints on the leading (leftmost) columns.

PostgreSQL automatically creates a unique index when a unique constraint or primary key is defined for a table. The index covers the columns that make up the primary key or unique constraint (a multicolumn index, if appropriate)

```
SELECT * FROM test1 WHERE lower(col1) = 'value';
CREATE UNIQUE INDEX test1_lower_col1_idx ON test1 (lower(col1));

SELECT * FROM people WHERE (first_name || ' ' || last_name) = 'John Smith';
CREATE INDEX people_names ON people ((first_name || ' ' || last_name));
```

A partial index is an index built over a subset of a table. One major reason for using a partial index is to avoid indexing common values. This reduces the size of the index, which will speed up those queries that do use the index. It will also speed up many table update operations because the index does not need to be updated in all cases.

Suppose you are storing web server access logs in a database. Most accesses originate from the IP address range of your organization but some are from elsewhere (say, employees on dial-up connections). If your searches by IP are primarily for outside accesses,
```
CREATE TABLE access_log (
    url varchar,
    client_ip inet
);
CREATE INDEX access_log_client_ip_ix ON access_log (client_ip)
WHERE NOT (client_ip > inet '192.168.100.0' AND
           client_ip < inet '192.168.100.255');

SELECT *
FROM access_log
WHERE url = '/index.html' AND client_ip = inet '212.78.10.32'; -- will use partial index
```
If you have a table that contains both billed and unbilled orders, where the unbilled orders take up a small fraction of the total table and yet those are the most-accessed rows
```
CREATE INDEX orders_unbilled_index ON orders (order_nr)
    WHERE billed is not true;
SELECT * FROM orders WHERE billed is not true AND order_nr < 10000;
```
PostgreSQL supports partial indexes with arbitrary predicates, so long as only columns of the table being indexed are involved. However, keep in mind that the predicate must match the conditions used in the queries that are supposed to benefit from the index.

Keep in mind that setting up a partial index indicates that you know at least as much as the query planner knows, in particular you know when an index might be profitable. Forming this knowledge requires experience and understanding of how indexes in PostgreSQL work. In most cases, the advantage of a partial index over a regular index will be minimal.

index is stored separately from the table's main data area (which is called the table's heap). This means that in an ordinary index scan, each row retrieval requires fetching data from both the index and the heap. To solve this performance problem, PostgreSQL supports index-only scans, which can answer queries from an index alone without any heap access. The query must reference only columns stored in the index. For example, given an index on columns `x` and `y` of a table that also has a column `z`, 
```
SELECT x, y FROM tab WHERE x = 'key';
SELECT x FROM tab WHERE x = 'key' AND y < 42;
```
Since queries typically need to retrieve more columns than just the ones they search on, PostgreSQL allows you to create an index in which some columns are just “payload” and are not part of the search key
```
CREATE INDEX tab_x_y ON tab(x) INCLUDE (y);

-- handle these queries as index-only scans, because y can be obtained from the index without visiting the heap.
SELECT y FROM tab WHERE x = 'key';
```

Always run `ANALYZE` first. This command collects statistics about the distribution of the values in the table. This information is required to estimate the number of rows returned by a query, which is needed by the planner to assign realistic costs to each possible query plan. In absence of any real statistics, some default values are assumed, which are almost certain to be inaccurate. Examining an application's index usage without having run `ANALYZE` is therefore a lost cause.

https://www.postgresql.org/docs/current/textsearch-intro.html#TEXTSEARCH-INTRO

For searches within PostgreSQL, a document is normally a textual field within a row of a database table, or possibly a combination (concatenation) of such fields, perhaps stored in several tables or obtained dynamically. In other words, a document can be constructed from different parts for indexing and it might not be stored anywhere as a whole. 

For text search purposes, each document must be reduced to the preprocessed tsvector format. Searching and ranking are performed entirely on the tsvector representation of a document — the original text need only be retrieved when the document has been selected for display to a user. We therefore often speak of the tsvector as being the document

Full text searching in PostgreSQL is based on the match operator `@@` which returns `true` if a `tsvector` (document) matches a `tsquery` (query)
```
SELECT 'a fat cat sat on a mat and ate a fat rat'::tsvector @@ 'cat & rat'::tsquery; -- t
```
A tsquery contains search terms, which must be already-normalized lexemes, and may combine multiple terms using AND, OR, NOT, and FOLLOWED BY operators. (https://www.postgresql.org/docs/current/datatype-textsearch.html#DATATYPE-TSQUERY)

`to_tsquery, plainto_tsquery, and phraseto_tsquery` helpful in converting user-written text into a proper tsquery, primarily by normalizing words appearing in the text. Similarly, `to_tsvector` is used to parse and normalize a document string
```
SELECT to_tsvector('fat cats ate fat rats') @@ to_tsquery('fat & rat'); --t
SELECT 'fat cats ate fat rats'::tsvector @@ to_tsquery('fat & rat'); --f
```

`fat & ! rat` matches documents that contain `fat` but not `rat`. `<->` (FOLLOWED BY) tsquery operator, matches only if its arguments have matches that are adjacent and in the given order.
```
SELECT to_tsvector('fatal error') @@ to_tsquery('fatal <-> error'); --t
SELECT to_tsvector('error is not fatal') @@ to_tsquery('fatal <-> error'); --f
```
`<2>` allows exactly one other lexeme to appear between the matches, and so on. The `phraseto_tsquery` function makes use of this operator to construct a tsquery that can match a multi-word phrase when some of the words are stop words.
```
SELECT phraseto_tsquery('cats ate rats'); --  'cat' <-> 'ate' <-> 'rat'
SELECT phraseto_tsquery('the cats ate the rats'); --  'cat' <-> 'ate' <2> 'rat'
```
`!x <-> y` matches `y` if it is not immediately after an `x`.

```
SELECT title FROM pgweb
WHERE to_tsvector(body) @@ to_tsquery('friend'); --  row  contains `friend` in body
```
Practical use of text searching usually requires creating an index
```
-- WHERE to_tsvector('english', body) @@ 'a & b' can use the index
CREATE INDEX pgweb_idx ON pgweb USING GIN (to_tsvector('english', body));

-- WHERE to_tsvector(config_name, body) @@ 'a & b' can use the index
CREATE INDEX pgweb_idx ON pgweb USING GIN (to_tsvector(config_name, body)); -- config_name is column in pgweb

-- Indexes can even concatenate columns
CREATE INDEX pgweb_idx ON pgweb USING GIN (to_tsvector('english', title || ' ' || body));
```
Another approach is to create a separate `tsvector` column to hold the output of `to_tsvector`. To keep this column automatically up to date with its source data, use a stored generated column. This example is a concatenation of title and body, using `coalesce` to ensure that one field will still be indexed when the other is `NULL`:
```
ALTER TABLE pgweb
ADD COLUMN textsearchable_index_col tsvector
GENERATED ALWAYS AS (to_tsvector('english', coalesce(title, '') || ' ' || coalesce(body, ''))) STORED;
CREATE INDEX textsearch_idx ON pgweb USING GIN (textsearchable_index_col);

-- ten most recent documents that contain create and table in the title or body
SELECT title
FROM pgweb
WHERE textsearchable_index_col @@ to_tsquery('create & table')
ORDER BY last_mod_date DESC
LIMIT 10;
```
One advantage of the separate-column approach over an expression index is that it is not necessary to explicitly specify the text search configuration in queries in order to make use of the index. the query can depend on `default_text_search_config`. Another advantage is that searches will be faster, since it will not be necessary to redo the `to_tsvector` calls to verify index matches. (This is more important when using a GiST index than a GIN index;)

The function setweight can be used to label the entries of a `tsvector` with a given weight, where a weight is one of the letters `A, B, C, or D`. This is typically used to mark entries coming from different parts of a document, such as title versus body. Because `to_tsvector(NULL)` will return `NULL`, it is recommended to use `coalesce`.
```
UPDATE tt SET ti =
    setweight(to_tsvector(coalesce(title,'')), 'A')    ||
    setweight(to_tsvector(coalesce(keyword,'')), 'B')  ||
    setweight(to_tsvector(coalesce(abstract,'')), 'C') ||
    setweight(to_tsvector(coalesce(body,'')), 'D');
```
```
SELECT to_tsquery('english', 'The & Fat & Rats'); -- 'fat' & 'rat'
SELECT to_tsquery('english', 'Fat | Rats:AB'); -- 'fat' | 'rat':AB
SELECT to_tsquery('supern:*A & star:*AB'); -- 'supern':*A & 'star':*AB
SELECT plainto_tsquery('english', 'The Fat Rats'); -- 'fat' & 'rat'
```
`phraseto_tsquery` behaves much like `plainto_tsquery`, except that it inserts the `<->` operator between surviving words instead of the `&` operator

https://www.postgresql.org/docs/current/textsearch-controls.html#TEXTSEARCH-RANKING

https://www.postgresql.org/docs/current/textsearch-controls.html#TEXTSEARCH-HEADLINE

You can use the `EXPLAIN` command to see what query plan the planner creates for any query.

https://www.postgresql.org/docs/current/using-explain.html

https://www.postgresql.org/docs/current/planner-stats.html

https://www.postgresql.org/docs/current/populate.html
