
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

An index on a primary (data) file is called the primary index. However, in most cases we can also assume that the primary index is built over a primary key or a set of keys identified as primary. All other indexes are called secondary.

## B-Tree Basics
- tbd
  
## Buffer Management
To reduce the number of accesses to persistent storage, pages are cached in memory. When the page is requested again by the storage layer, its cached copy is returned. Cached pages available in memory can be reused under the assumption that no other process has modified the data on disk. This approach is sometimes referenced as virtual disk [BAYER72]. A virtual disk read accesses physical storage only if no copy of the page is already available in memory. A more common name for the same concept is page cache or buffer pool. The page cache is responsible for caching pages read from disk in memory. In case of a database system crash or unorderly shutdown, cached contents are lost.

We can “lock” pages that have a high probability of being used in the nearest time. Locking pages in the cache is called pinning. Pinned pages are kept in memory for a longer time, which helps to reduce the number of disk accesses and improve performance

## Log Semantics
The write-ahead log is append-only and its written contents are immutable, so all writes to the log are sequential. Since the WAL is an immutable, append-only data structure, readers can safely access its contents up to the latest write threshold while the writer continues appending data to the log tail. The WAL consists of log records. Every record has a unique, monotonically increasing log sequence number (LSN). Usually, the LSN is represented by an internal counter or a timestamp. Since log records do not necessarily occupy an entire disk block, their contents are cached in the log buffer and are flushed on disk in a force  operation. Forces happen as the log buffers fill up, and can be requested by the transaction manager or a page cache. All log records have to be flushed on disk in LSN order. Besides individual operation records, the WAL holds records indicating transaction completion. A transaction can’t be considered committed until the log is forced up to the LSN of its commit record. To make sure the system can continue functioning correctly after a crash during rollback or recovery, some systems use compensation log records (CLR) during undo and store them in the log.

## Concurrency Control
Concurrency control is a set of techniques for handling interactions between concurrently executing transactions. These techniques can be roughly grouped into the following categories:
- Optimistic concurrency control (OCC)
  Allows transactions to execute concurrent read and write operations, and determines whether or not the result of the combined execution is serializable. In other words, transactions do not block each other,
maintain histories of their operations, and check these histories for possible conflicts before commit. If execution results in a conflict, one of the conflicting transactions is aborted.
- Multiversion concurrency control (MVCC)
  Guarantees a consistent view of the database at some point in the past identified by the timestamp by allowing multiple timestamped versions of the record to be present. MVCC can be implemented using validation techniques, allowing only one of the updating or committing transactions to win, as well as with lockless techniques such as timestamp ordering, or lock-based ones, such as two-phase locking.
- Pessimistic (also known as conservative) concurrency control (PCC)
    There are both lock-based and nonlocking conservative methods, which differ in how they manage and grant access to shared resources. Lock-based approaches require transactions to maintain locks on database records to prevent other transactions from modifying locked records and assessing records that are being modified until the transaction releases its locks. Nonlocking approaches maintain read and write operation lists and restrict execution, depending on the schedule of unfinished transactions.

- read uncommitted
    allow dirty, non-repeatable and phantom read
- read committed
    allow non-repeatable and phantom read
- repeatable read
    allow phantom read
- serializable


### Optimistic Concurrency Control
Optimistic concurrency control assumes that transaction conflicts occur rarely and, instead of using locks and blocking transaction execution, we can validate transactions to prevent read/write conflicts with concurrently executing transactions and ensure serializability before committing their results.
Generally, transaction execution is split into three phases: 
- Read phase
  
    The transaction executes its steps in its own private context, without making any of the changes visible to other transactions

- Validation phase

    Read and write sets of concurrent transactions are checked for the presence of possible conflicts between their operations that might violate serializability. If some of the data the transaction was reading
is now out-of-date, or it would overwrite some of the values written by transactions that committed during its read phase, its private context is cleared and the read phase is restarted. In other words, the
validation phase determines whether or not committing the transaction preserves ACID properties.

- Write phase
  
    If the validation phase hasn’t determined any conflicts, the transaction can commit its write set from the private context to the database state.

