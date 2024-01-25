
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
Files are organized in a way that minimizes storage overhead per
stored data record.
- Access efficiency
Records can be located in the smallest possible number of steps.
- Update efficiency
Record updates are performed in a way that minimizes the number of
changes on disk.
