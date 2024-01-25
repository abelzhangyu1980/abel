
# storage engine

## column vs row-oriented DBMS

Since row-oriented stores are most useful in scenarios when we have to access data by row, storing entire rows together improves spatial locality.
Because data on a persistent medium such as a disk is typically accessed block-wise (in other words, a minimal unit of disk access is a block), a single block will contain data for all columns. This is great for cases when

we’d like to access an entire user record, but makes queries accessing individual fields of multiple user records (for example, queries fetching only the phone numbers) more expensive, since data for the other fields
will be paged in as well.

Column-oriented database management systems partition data vertically (i.e., by column) instead of storing it in rows. Here, values for the same column are stored contiguously on disk (as opposed to storing rows
contiguously as in the previous example). For example, if we store historical stock market prices, price quotes are stored together. Storing values for different columns in separate files or file segments allows
efficient queries by column, since they can be read in one pass rather than consuming entire rows and discarding data for columns that weren’t queried.
Column-oriented stores are a good fit for analytical workloads that compute aggregates, such as finding trends, computing average values, etc.

## Data Files and Index Files
- Storage efficiency

Files are organized in a way that minimizes storage overhead per stored data record.

- Access efficiency

Records can be located in the smallest possible number of steps. 

- Update efficiency

Record updates are performed in a way that minimizes the number of changes on disk.


### Data Files

Data files (sometimes called primary files) can be implemented as indexorganized tables (IOT), heap-organized tables (heap files), or hashorganized tables (hashed files).

- Records in heap files are not required to follow any particular order, and most of the time they are placed in a write order. This way, no additional work or file reorganization is required when new pages are appended. Heap files require additional index structures, pointing to the locations where data records are stored, to make them searchable.
- In hashed files, records are stored in buckets, and the hash value of the key determines which bucket a record belongs to. Records in the bucket can be stored in append order or sorted by key to improve lookup speed.
- Index-organized tables (IOTs) store data records in the index itself. Since records are stored in key order, range scans in IOTs can be implemented by sequentially scanning its contents. Storing data records in the index allows us to reduce the number of disk seeks by at least one, since after traversing the index and locating the searched key, we do not have to address a separate file to find the associated data record.

### Index Files
An index is a structure that organizes data records on disk in a way that facilitates efficient retrieval operations. Index files are organized as specialized structures that map keys to locations in data files where the records identified by these keys (in the case of heap files) or primary keys (in the case of index-organized tables) are stored.
