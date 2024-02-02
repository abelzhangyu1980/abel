# Anatomy of an Index

An index is a distinct structure in the database that is built using the create index statement. It requires its own disk space and holds a copy of the indexed table data. That means that an index is pure redundancy. Creating an index does not change the table data; it just creates a new data structure that refers to the table. A database index is, after all, very much like the index at the end of a book: it occupies its own space, it is highly redundant, and it refers to the actual information stored in a different place.  Searching in a database index is like searching in a printed telephone directory. The key concept is that all entries are arranged in a well-defined order. Finding data in an ordered data set is fast and easy because the sort order determines each entries position
A database index is, however, more complex than a printed directory because it undergoes constant change. Updating a printed directory for every change is impossible for the simple reason that there is no space between existing entries to add new ones. A printed directory bypasses this problem by only handling the accumulated updates with the next printing. An SQL database cannot wait that long. It must process insert, delete and update statements immediately, keeping the index order without moving large amounts of data. The database combines two data structures to meet the challenge: a doubly linked list and a search tree. These two structures explain most of the database’s performance characteristics.

## Leaf nodes - double linked list
The primary purpose of an index is to provide an ordered representation of the indexed data. It is, however, not possible to store the data sequentially because an insert statement would need to move the following entries to make room for the new one. Moving large amounts of data is very timeconsuming so the insert statement would be very slow. The solution to the problem is to establish a logical order that is independent of physical order in memory. The logical order is established via a doubly linked list. Every node has links to two neighboring entries, very much like a chain. New nodes are inserted between two existing nodes by updating their links to refer to the new node. The physical location of the new node doesn’t matter because the doubly linked list maintains the logical order. 
The data structure is called a doubly linked list because each node refers to the preceding and the following node. It enables the database to read the index forwards or backwards as needed. It is thus possible to insert new entries without moving large amounts of data — it just needs to change some pointers.

## Searcg Tree (B-Tree)
balanced tree provides very efficient traversal.

An index lookup requires three steps: (1) the tree traversal; (2) following the leaf node chain; (3) fetching the table data. The tree traversal is the only step that has an upper bound for the number of accessed blocks — the index depth. The other two steps might need to access many blocks — they cause a slow index lookup.

Logarithmic Scalability

- INDEX UNIQUE SCAN The INDEX UNIQUE SCAN performs the tree traversal only. The Oracle database uses this operation if a unique constraint ensures that the search criteria will match no more than one entry
- INDEX RANGE SCAN The INDEX RANGE SCAN performs the tree traversal and follows the leaf node chain to find all matching entries This is the fallback operation if multiple entries could possibly match the search criteria.
- TABLE ACCESS BY INDEX ROWID The TABLE ACCESS BY INDEX ROWID operation retrieves the row from the table. This operation is (often) performed for every matched record from a preceding index scan operation

# The Where Clause
The where clause defines the search condition of an SQL statement, and it thus falls into the core functional domain of an index: finding data quickly.

## The Equality Operator

SELECT first_name, last_name FROM employees WHERE employee_id = 123
The database automatically creates an index for the primary key. The where clause cannot match multiple rows because the primary key constraint ensures uniqueness of the EMPLOYEE_ID values
After accessing the index, the database must do one more step to fetch the queried data (FIRST_NAME, LAST_NAME) from the table storage: the TABLE ACCESS BY INDEX ROWID operation. This operation can become a performance bottleneck — as explained in “Slow Indexes, Part I” — but there is no such risk in connection with an INDEX UNIQUE SCAN. This operation cannot deliver more than one entry so it cannot trigger more than one table access. That means that the ingredients of a slow query are not present with an INDEX UNIQUE SCAN. 

## Concatenated Indexes

Even though the database creates the index for the primary key automatically, there is still room for manual refinements if the key consists of multiple columns. In that case the database creates an index on all primary key columns — a so-called concatenated index (also known as multicolumn, composite or combined index). Note that the column order of a concatenated index has great impact on its usability so it must be chosen carefully. 
CREATE UNIQUE INDEX employee_pk ON employees (employee_id, subsidiary_id);

A query for a particular employee has to take the full primary key into account — that is, the SUBSIDIARY_ID column also has to be used: 
SELECT first_name, last_name FROM employees WHERE employee_id = 123 AND subsidiary_id = 30;

Whenever a query uses the complete primary key, the database can use an INDEX UNIQUE SCAN — no matter how many columns the index has. But what happens when using only one of the key columns, for example, when searching all employees of a subsidiary?
SELECT first_name, last_name FROM employees WHERE subsidiary_id = 20;

The execution plan reveals that the database does not use the index. Instead it performs a FULL TABLE SCAN. As a result the database reads the entire table and evaluates every row against the where clause. The execution time grows with the table size: if the table grows tenfold, the FULL TABLE SCAN takes ten times as long. The database does not use the index because it cannot use single columns from a concatenated index arbitrarily. A closer look at the index structure. 
makes this clear.
The most important consideration when defining a concatenated index is how to choose the column order so it can be used as often as possible.
In general, a database can use a concatenated index when searching with the leading (leftmost) columns. An index with three columns can be used when searching for the first column, when searching with the first two columns together, and when searching using all  columns.
Even though the two-index solution delivers very good select performance as well, the single-index solution is preferable. It not only saves storage space, but also the maintenance overhead for the second index. The fewer indexes a table has, the better the  insert, delete and update performance.

### Full Table Scan
The operation TABLE ACCESS FULL, also known as full table scan, can be the most efficient operation in some cases anyway, in particular when retrieving a large part of the table. This is partly due to the overhead for the index lookup itself, which does not happen for a TABLE ACCESS FULL operation. This is mostly because an index lookup reads one block after the other as the  database does not know which block to read next until the current block has been processed. A FULL TABLE SCAN must get the entire table anyway so that the database can read larger chunks at a time (multi block read). Although the database reads more data, it might need to execute fewer read operations.  

# Slow Indexes, Part II

## The Query Optimizer
The query optimizer, or query planner, is the database component that transforms an SQL statement into an execution plan. This process is also called compiling or parsing. There are two distinct optimizer types.
Cost-based optimizers (CBO) generate many execution plan variations and calculate a cost value for each plan. The cost calculation is based on the operations in use and the estimated row numbers. In the end the cost value serves as the benchmark for picking the “best” execution plan.
Rule-based optimizers (RBO) generate the execution plan using a hardcoded rule set. Rule based optimizers are less flexible and are seldom used today.

SELECT /*+ NO_INDEX(EMPLOYEES EMPLOYEE_PK) */
first_name, last_name, subsidiary_id, phone_number FROM employees WHERE last_name = 'WINAND' AND subsidiary_id = 30;

The execution plan that was presumably used before the index change did
not use an index at all:
----------------------------------------------------
| Id | Operation | Name | Rows | Cost |
----------------------------------------------------
| 0 | SELECT STATEMENT | | 1 | 477 |
|* 1 | TABLE ACCESS FULL| EMPLOYEES | 1 | 477 |
----------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------
1 - filter("LAST_NAME"='WINAND' AND "SUBSIDIARY_ID"=30)

Even though the TABLE ACCESS FULL must read and process the entire table, it seems to be faster than using the index in this case. That is particularly unusual because the query matches one row only. Using an index to find a single row should be much faster than a full table scan, but in this case it is not. The index seems to be slow.

In such cases it is best to go through each step of the troublesome execution plan. The first step is the INDEX RANGE SCAN on the EMPLOYEES_PK index. That index does not cover the LAST_NAME column — the INDEX RANGE SCAN can consider the SUBSIDIARY_ID filter only; the Oracle database shows this in the “Predicate Information” area — entry “2” of the execution plan. There you can see the conditions that are applied for each operation.
The INDEX RANGE SCAN with operation ID 2 (Example 2.1 on page 19) applies only the SUBSIDIARY_ID=30 filter. That means that it traverses the index tree to find the first entry for SUBSIDIARY_ID 30. Next it follows the leaf node chain to find all other entries for that subsidiary. The result of the INDEX RANGE SCAN is a list of ROWIDs that fulfill the SUBSIDIARY_ID condition: depending on the subsidiary size, there might be just a few ones or there could be many hundreds. The next step is the TABLE ACCESS BY INDEX ROWID operation. It uses the ROWIDs from the previous step to fetch the rows — all columns — from the table. Once the LAST_NAME column is available, the database can evaluate the remaining part of the where clause. That means the database has to fetch all rows for SUBSIDIARY_ID=30 before it can apply the LAST_NAME filter
The statement’s response time does not depend on the result set size but on the number of employees in the particular subsidiary. If the subsidiary has just a few members, the INDEX RANGE SCAN provides better performance. Nonetheless a TABLE ACCESS FULL can be faster for a huge subsidiary because it can read large parts from the table in one shot (see “Full Table Scan” on page 13). The query is slow because the index lookup returns many ROWIDs — one for each employee of the original company — and the database must fetch them individually. It is the perfect combination of the two ingredients that make an index slow: the database reads a wide index range and has to fetch many rows individually.

Choosing the best execution plan depends on the table’s data distribution as well so the optimizer uses statistics about the contents of the database. In our example, a histogram containing the distribution of employees over subsidiaries is used. This allows the optimizer to estimate the number of rows returned from the index lookup — the result is used for the cost calculation.

## Statistics
A cost-based optimizer uses statistics about tables, columns, and indexes. Most statistics are collected on the column level: the number of distinct values, the smallest and largest values (data range), the number of NULL occurrences and the column histogram (data
distribution). The most important statistical value for a table is its size (in rows and blocks). The most important index statistics are the tree depth, the number of leaf nodes, the number of distinct keys and the clustering factor (see Chapter 5, “Clustering Data”). The optimizer uses these values to estimate the selectivity of the where clause predicates.

## Over-Indexing

## Index Merge
It is one of the most common question about indexing: is it better to create one index for each column or a single index for all columns of a where clause? The answer is very simple in most cases: one index with multiple columns is better.

Nevertheless there are queries where a single index cannot do a perfect job, no matter how you define the index; e.g., queries with two or more independent range conditions as in the following example: SELECT first_name, last_name, date_of_birth FROM employees WHERE UPPER(last_name) < ? AND date_of_birth < ? It is impossible to define a B-tree index that would support this query without filter predicates. For an explanation, you just need to remember that an index is a linked list. If you define the index as  PPER(LAST_NAME), DATE_OF_BIRTH (in that order), the list begins with A and ends with Z. The date of birth is considered only when there are two employees with the same name. If you define the index the other way around, it will start with the eldest employees  and end with the youngest. In that case, the names only have a minor impact on the sort order. No matter how you twist and turn the index definition, the entries are always arranged along a chain. At one end, you have the small entries and at the other end the big ones. An index can therefore only support one range condition as an access predicate.

The other option is to use two separate indexes, one for each column. Then the database must scan both indexes first and then combine the results. The duplicate index lookup alone already involves more effort because the database has to traverse two index  trees. Additionally, the database needs a lot of memory and CPU time to combine the intermediate results

## Partial index
A partial index is useful for commonly used where conditions that use constant values — like the status code in the following example:

SELECT message FROM messages WHERE processed = 'N' AND receiver = ?

Queries like this are very common in queuing systems. The query fetches all unprocessed messages for a specific recipient. Messages that were already processed are rarely needed. If they are needed, they are usually accessed by a more specific criteria like the primary key. We can optimize this query with a two-column index. Considering this query only, the column order does not matter because there is no range condition. 

CREATE INDEX messages_todo  ON messages (receiver, processed) 

The index fulfills its purpose, but it includes many rows that are never searched, namely all the messages that were already processed. Due to the logarithmic scalability the index nevertheless makes the query very fast even though it wastes a lot of disk space.

With partial indexing you can limit the index to include only the unprocessed messages. The syntax for this is surprisingly simple: a where clause.

CREATE INDEX messages_todo ON messages (receiver) WHERE processed = 'N'

# The Join Operation
There is, however, one thing that is common to all join algorithms: they process only two tables at a time. A SQL query with more tables requires multiple steps: first building an intermediate result set by joining two tables, then joining the result with the next table and so forth.

## Nested Loops
The nested loops join is the most fundamental join algorithm. It works like using two nested queries: the outer or driving query to fetch the results from one table and a second query for each row from the driving query to fetch the corresponding data from the other table.

## Hash Join
The hash join algorithm aims for the weak spot of the nested loops join: the many B-tree traversals when executing the inner query. Instead it loads the candidate records from one side of the join into a hash table that can be probed very quickly for each row from the other side of the join. Tuning a hash join requires an entirely different indexing approach than the nested loops join. Beyond that, it is also possible to improve hash join performance by selecting fewer columns — a challenge for most ORM tools.
The indexing strategy for a hash join is very different because there is no need to index the join columns. Only indexes for independent where predicates improve hash join performance.

# Clustering Data

Clustering data means to store consecutively accessed data closely together so that accessing it requires fewer IO operations. Data clusters are very important in terms of database tuning.
The simplest data cluster in an SQL database is the row. Databases store all columns of a row in the same database block if possible. Exceptions apply if a row doesn’t fit into a single block — e.g., when LOB types are involved

## Index Filter Predicates Used Intentionally

Where clause predicates that cannot serve as access predicate are good candidates for this technique:
SELECT first_name, last_name, subsidiary_id, phone_number
FROM employees
WHERE subsidiary_id = ?
AND UPPER(last_name) LIKE '%INA%';

Remember that LIKE expressions with leading wildcards cannot use the index tree. That means that indexing LAST_NAME doesn’t narrow the scanned index range — no matter if you index LAST_NAME or UPPER(last_name). This condition is therefore no good candidate for indexing.
However the condition on SUBSIDIARY_ID is well suited for indexing. We don’t even need to add a new index because the SUBSIDIARY_ID is already the leading column in the index for the primary key.

--------------------------------------------------------------
|Id | Operation | Name | Rows | Cost |
--------------------------------------------------------------
| 0 | SELECT STATEMENT | | 17 | 230 |
|*1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES | 17 | 230 |
|*2 | INDEX RANGE SCAN | EMPLOYEE_PK| 333 | 2 |
--------------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------
1 - filter(UPPER("LAST_NAME") LIKE '%INA%')
2 - access("SUBSIDIARY_ID"=TO_NUMBER(:A))

In the above execution plan, the cost value raises a hundred times from the INDEX RANGE SCAN to the subsequent TABLE ACCESS BY INDEX ROWID operation. In other words: the table access causes the most work. It is actually a common pattern and is not a problem by itself. Nevertheless, it is the most significant contributor to the overall execution time of this query

The table access is not necessarily a bottleneck if the accessed rows are stored in a single table block because the database can fetch all rows with a single read operation. If the same rows are spread across many different blocks, in contrast, the table access can become a serious performance problem because the database has to fetch many blocks in order to retrieve all the rows. That means the performance depends on the physical distribution of the accessed rows — in other words: it depends on the clustering of rows.

This is exactly where the second power of indexing — clustering data — comes in. You can add many columns to an index so that they are automatically stored in a well defined order. That makes an index a powerful yet simple tool for clustering data.

To apply this concept to the above query, we must extend the index to cover all columns from the where clause — even if they do not narrow the scanned index range:
**CREATE INDEX empsubupnam ON employees (subsidiary_id, UPPER(last_name));**

The column SUBSIDIARY_ID is the first index column so it can be used as an access predicate. The expression UPPER(last_name) covers the LIKE filter as index filter predicate. Indexing the uppercase representation saves a few CPU cycles during execution, but a straight index on LAST_NAME would work as well. You’ll find more about this in the next section.
--------------------------------------------------------------
|Id | Operation | Name | Rows | Cost |
--------------------------------------------------------------
| 0 | SELECT STATEMENT | | 17 | 20 |
| 1 | TABLE ACCESS BY INDEX ROWID| EMPLOYEES | 17 | 20 |
|*2 | INDEX RANGE SCAN | EMPSUBUPNAM| 17 | 3 |
--------------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------
2 - access("SUBSIDIARY_ID"=TO_NUMBER(:A))
filter(UPPER("LAST_NAME") LIKE '%INA%')

The new execution plan shows the very same operations as before. The cost value dropped considerably nonetheless. In the predicate information we can see that the LIKE filter is already applied during the INDEX RANGE SCAN. Rows that do not fulfill the LIKE filter are immediately discarded. The table access does not have any filter predicates anymore. That means it does not load rows that do not fulfill the where clause.

The difference between the two execution plans is clearly visible in the “Rows” column. According to the optimizer’s estimate, the query ultimately matches 17 records. The index scan in the first execution plan delivers 333 rows nevertheless. The database must then load these 333 rows from the table to apply the LIKE filter which reduces the result to 17 rows. In the second execution plan, the index access does not deliver those rows in the first place so the database needs to execute the TABLE ACCESS BY INDEX ROWID operation only 17 times.

## Index-Only Scan
The index-only scan is one of the most powerful tuning methods of all. It not only avoids accessing the table to evaluate the where clause, but avoids accessing the table completely if the database can find the selected columns in the index itself.

To cover an entire query, an index must contain all columns from the SQL statement — in particular also the columns from the select clause as shown in the following example:
CREATE INDEX sales_sub_eur ON sales ( subsidiary_id, eur_value ); 

SELECT SUM(eur_value) FROM sales WHERE subsidiary_id = ?; 

Of course indexing the where clause takes precedence over the other clauses. The column SUBSIDIARY_ID is therefore in the first position so it qualifies as an access predicate.

The execution plan shows the index scan without a subsequent table access
(TABLE ACCESS BY INDEX ROWID).
----------------------------------------------------------
| Id | Operation | Name | Rows | Cost |
----------------------------------------------------------
| 0 | SELECT STATEMENT | | 1 | 104 |
| 1 | SORT AGGREGATE | | 1 | |
|* 2 | INDEX RANGE SCAN| SALES_SUB_EUR | 40388 | 104 |
----------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------
2 - access("SUBSIDIARY_ID"=TO_NUMBER(:A))
The index covers the entire query so it is also called a covering index.

The index has a copy of the EUR_VALUE column so the database can use the value stored in the index. Accessing the table is not required because the index has all of the information to satisfy the query.







