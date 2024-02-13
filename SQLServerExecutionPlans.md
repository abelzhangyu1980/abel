# Clustered index
index leap contains data... one clustered index for one table as data can only be stored and sorted once. 

# Index Scan

Indexes in SQL Server are stored in a balanced-tree, or a b-tree (a series of nodes that point to a parent). A clustered index not only stores the key structure, like a regular index, but also sorts and stores the data at the lowest level of the index, known as the leaf, which is the reason why there can be only one clustered index per table. This means that a Clustered Index Scan is very similar in concept to a Table Scan. The entire index, or a large percentage of it, is being traversed, row by row, in order to retrieve the data needed by the query.

An Index Scan often occurs, as in this case, when an index exists but the optimizer determines that there are so many rows to return that it is quicker to simply scan all the values in the index rather than use the keys provided by that index. In other situations, a scan is necessary because the index is not selective enough for the optimizer to be sure of finding the values it needs without scanning a large percentage of the index. That can also occur when the statistics for that index are out of date and showing incorrect information. You can also have situations where a query applies functions to columns, which means the optimizer can't determine what the value for that column could be so it has to scan the entire index to find it.

An obvious question to ask, if you see an Index Scan in your execution plan, is whether you are returning more rows than is necessary. If the number of rows returned is higher than you expect, that's a strong indication that you need to fine-tune the WHERE clause of your query so that only those rows that are actually needed are returned. Returning unnecessary rows wastes SQL Server resources and hurts overall performance.

In a scan operation, SQL Server navigates down to the first or last leaf-level page of the index and then scans forward or backward through the leaf pages. A scan often reads all the pages in the leaf level of the index, but may read only a portion of the index in some cases.

A scan often occurs when all rows need to be read to satisfy the definition of the query. You can also see a scan when so many rows need to be read that scanning them all would take less time than navigating the index structure to find them (a.k.a. "seeking," discussed shortly). Sometimes, the optimizer chooses a scan because there is no usable index for the Predicate columns, or because the query is written in such a way that performing a seek against the index is not possible (for example, a function against a column will lead to scans).



# Clustered Index Seek

A Clustered Index Seek operator occurs when a query uses the index to access only one row, or a few contiguous rows. It's one of the faster ways to retrieve data from the system. We can easily make the previous query more efficient by adding a WHERE clause. A WHERE
clause limits the amount of data being returned, which makes it more likely that indexes will be used to return the appropriate data.

Index seeks are completely different from scans, where the engine walks through all the rows to find what it needs. Clustered and NonClustered Index Seeks occur when the optimizer is able to locate an index that it can use to retrieve the required records.
Therefore, it tells the storage engine to look up the values based on the keys of the given index by assigning the Seek operation instead of a scan.

When an index is used in a Seek operation, the key values are used to look up and quickly identify the row or rows of data needed. This is similar to looking up a word in the index of a book to get the correct page number. The benefit of the Clustered Index Seek is that,
not only is the Index Seek usually an inexpensive operation when compared to a scan, but no extra steps are required to get the data because it is stored in the index, at the leaf level.

# NonClustered Index Seek

A NonClustered Index Seek operation is a seek against a non-clustered index. This operation is effectively no different than the Clustered Index Seek but the only data available is that which is stored in the index itself.
Like a Clustered Index Seek, a NonClustered Index Seek uses an index to look up the required rows. Unlike a Clustered Index Seek, a NonClustered Index Seek has to use a non-clustered index to perform the operation. Depending on the query and index, the query optimizer might be able to find all the data in the non-clustered index. 
However, a non-clustered index only stores the key values; it doesn't store the data. The optimizer might have to look up the data in the clustered index, slightly hurting performance due to the additional I/O required to perform the extra lookups – more on this in the next section.
If the index satisfies all the needs of the query, meaning all the data needed is stored with the index, in the key, which makes this index, in this situation, a covering index

# Key Lookup

A Key Lookup operator (there are two, RID and Key) is required to get data from the heap or the clustered index, respectively, when a non-clustered index is used, but is not a covering index

SELECT p.BusinessEntityID, p.LastName, p.FirstName, p.NameStyle FROM Person.Person AS p WHERE p.LastName LIKE 'Jaf%';

 ![key lookup](/keylookup.jpg)

 Finally, we get to see a plan that involves more than a single operation! Reading the plan in the physical order from right to left and top to bottom, the first operation we see is an Index Seek against the IX_Person_LastName_FirstName_MiddleName index. This
is a non-unique, non-clustered index and, in the case of this query, it is non-covering. A covering index is a non-clustered index that contains all of the columns that need to be referenced by a query, including columns in the SELECT list, JOIN criteria and the
WHERE clause.

In this case, the index on the Lastname and FirstName columns provides a quick way to retrieve information based on the filtering criteria LIKE 'Jaf%'. However, since only the name columns and the clustered index key are stored with the non-clustered index,
the other column being referenced, NameStyle, has to be pulled in from somewhere else.

Since this index is not a covering index, the query optimizer is forced to not only read the non-clustered index, but also to read the clustered index to gather all the data required to process the query. This is a Key Lookup and, essentially, it means that the optimizer
cannot retrieve the rows in a single operation, and has to use a clustered key (or a row ID from a heap table) to return the corresponding rows from a clustered index (or from the table itself).

We can understand the operations of the Key Lookup by using the information contained within the execution plan. If you first open the ToolTips window for the Index Seek operator by hovering with the mouse over that operator, you will see something similar to
Figure 2.8. The pieces of information in which we're now interested are the Output List and the Seek Predicates, down near the bottom of the window.

 ![key lookup](/indexseek.png)

 Several of the columns are returned from the index, Person.BusinessEntityID, Person.FirstName, and Person.LastName. In addition, you can see how the optimizer can modify a statement. If you look at the Seek Predicates, instead of a LIKE 'Jaf%', as was passed in the query, the optimizer has modified the statement
so that the actual predicate used is Person.LastName >= 'Jaf' and Person.LastName < 'JaG' (minus a bit of operational code).

A join operation, which combines the results of the two operations, always accompanies a Key Lookup. In this instance, it was a Nested Loops join operation

 ![key lookup](/nestedloops.png)

Typically, depending on the amount of data involved, a Nested Loops join by itself does not indicate any performance issues. In this case, because a Key Lookup operation is required, the Nested Loops join is needed to combine the rows of the Index Seek and Key
Lookup. If the Key Lookup was not needed (because a covering index was available), then the Nested Loops operator would not be needed in this execution plan. However, because this operator was involved, along with the Key Lookup operator, at least two additional
operations are required for every row returned from the non-clustered index. This is what can make a Key Lookup operation a very expensive process in terms of performance.

If this table had been a heap, a table without a clustered index, the operator would have been a RID Lookup operator. RID stands for row identifier, the means by which rows in a heap table are uniquely marked and stored within a table. The basics of the operation of a
RID Lookup are the same as a Key Lookup.

# Table Scan

A Table Scan can occur for several reasons, but it's often because there are no useful indexes on the table, and the query optimizer has to search through every row in order to identify the rows to return. Another common cause of a Table Scan is a query that
requests all the rows of a table

When all (or the majority) of the rows of a table are returned then, whether an index exists or not, it is often faster for the query optimizer to scan through each row and return them than look up each row in an index. This commonly occurs in tables with
few rows. 

Assuming that the number of rows in a table is relatively small, Table Scans are generally not a problem. On the other hand, if the table is large and many rows are returned, then you might want to investigate ways to rewrite the query to return fewer rows, or add an
appropriate index to speed performance.

# RID Lookup

RID Lookup is the heap equivalent of the Key Lookup operation. As was mentioned before, non-clustered indexes don't always have all the data needed to satisfy a query. When they do not, an additional operation is required to get that data. When there is a
clustered index on the table, it uses a Key Lookup operator as described above. When there is no clustered index, the table is a heap and must look up data using an internal identifier known as the Row ID or RID.

SELECT * FROM [dbo].[DatabaseLog] WHERE DatabaseLogID = 1 

 ![key lookup](/ridlookup.png)

 # Table Joins

 SELECT e.JobTitle, a.City, p.LastName + ', ' + p.FirstName AS EmployeeName FROM HumanResources.Employee AS e JOIN Person.BusinessEntityAddress AS bea ON e.BusinessEntityID = bea.BusinessEntityID JOIN Person.Address a
 ON bea.AddressID = a.AddressID JOIN Person.Person AS p ON e.BusinessEntityID = p.BusinessEntityID ; 

Associated with each operator icon, and displayed below it, is an estimated cost, assigned by the optimizer. From the relative estimated cost displayed, we can identify the three most costly operations in the plan, in descending order.
- The Index Scan against the Person.Address table (31%).
- The Hash Match join operation between the Person.Person table and the output from the first Hash Match (21%).
- The other Hash Match join operator between the Person.Address table and the output from the Nested Loop operator (20%).

 ![table join](/tablejoin.png)


# Hash Match join

A Hash Match operator appears in the plan when SQL Server puts two data sets into temporary tables, hash tables, and then uses these structures to compare data and arrive at the matching set.
Listing 2.7 had two different Hash Match join operations. Reading the plan logically, the first operation after the Select operator is a Hash Match join operation. This join is combining the output of one of the Index Scans with the combined output of the rest of the operations in the query. This Hash Match join operation is the second most expensive operation of this execution plan, so we would be interested in what we could
do to improve its performance. Figure 2.17 shows the properties for this operator.

 ![table join](/hashmatch.png)

 Before we can talk about what a Hash Match join is, we need to understand two new concepts: hashing and a hash table. Hashing is a programmatic technique where data is converted into a symbolic form that makes searching for that data much more efficient. For example, SQL Server programmatically converts a row of data in a table into a unique value that represents the contents of the row. In many ways, it is like taking a
row of data and encrypting it. Like encryption, a hashed value can be converted back to the original data.
A hash table, on the other hand, is a data structure that divides all of the elements into equal-sized categories, or buckets, to allow quick access to the elements. The hashing function determines into which bucket an element goes. For example, SQL Server can take a row from a table, hash it into a hash value, and then store the hash value in a hash table, in tempdb.

Now that we understand these terms, we can discuss the Hash Match join operator. It occurs when SQL Server has to join two large data sets, and decides to do so by first hashing the rows from the smaller of the two data sets, and inserting them into a hash table. It then processes the larger data set, one row at a time, against the hash table, looking for matches, indicating the rows to be joined.

The smaller of the data sets provides the values in the hash table, so the table size is small, and because hashed values instead of real values are used, comparisons are quick. As long as the hash table is relatively small, this can be a quick process. On the other hand, if both tables are very large, a Hash Match join can be very inefficient as compared to other types of joins. All the data for the hash table is stored within tempdb, so excessive use of Hash Joins in your queries can lead to a heavier load on tempdb.

# Nested Loops Join

A Nested Loops join functions by taking a set of data, referred to as the outer set, and comparing it, one row at a time to another set of data, called the inner set. This sounds like a cursor, and effectively, it is one but, with the appropriate data set, it can be a very efficient operation.

# Compute Scalar

This is simply a representation of an operation to produce a scalar, a single defined value, usually from a calculation – in this case, the alias EmployeeName, which combines the columns Contact.LastName and Contact.FirstName with a comma between them. While this was not a zero-cost operation (0.001997), the cost is trivial enough in the context of the query to be essentially free.

# Merge Join

JoinA Merge Join operator works from sorted data, and sorted data only. It takes the data from two different data sets and uses the fact that the data is sorted to simply merge it together, combining based on the matching values, which it can do very easily because the order of the values will be identical. If the data is sorted, this can be one of the most efficient join operations. However, the data is frequently not sorted, so sorting it for a Merge Join requires the addition of a Sort operator to ensure it works; this can make this join operation less efficient.

**SET SHOWPLAN_ALL ON;**
GO
SELECT e.BusinessEntityID , e.JobTitle , e.LoginID FROM HumanResources.Employee AS e WHERE e.LoginID = 'adventure-works\marc0';
GO
**SET SHOWPLAN_ALL OFF;**



