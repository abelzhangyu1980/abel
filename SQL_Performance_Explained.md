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

The execution plan reveals that the database does not use the index. Instead it performs a FULL TABLE SCAN. As a result the database reads the entire table and evaluates every row against the where clause. The execution time grows with the table size: if the table grows tenfold, the FULL TABLE SCAN takes ten times as long.

### Full Table Scan
The operation TABLE ACCESS FULL, also known as full table scan, can be the most efficient operation in some cases anyway, in particular when retrieving a large part of the table. This is partly due to the overhead for the index lookup itself, which does not happen for a TABLE ACCESS FULL operation. This is mostly because an index lookup reads one block after the other as the  database does not know which block to read next until the current block has been processed. A FULL TABLE SCAN must get the entire table anyway so that the database can read larger chunks at a time (multi block read). Although the database reads more data, it might need to execute fewer read operations.  





