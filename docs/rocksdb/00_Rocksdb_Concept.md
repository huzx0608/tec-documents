# Concept about Rocksdb

## Memtables

Three memtables are part of the library: a skiplist memtable, a vector memtable and a prefix-hash memtable.

- A skiplist memtable. The skiplist is a sorted set, which is a necessary construct when the workload interleaves writes with range-scans. Some applications do not interleave writes and scans, however, and some applications do not do range-scans at all.
- A vector memtable is appropriate for bulk-loading data into the database. Every write inserts a new element at the end of the vector; when it is time to flush the memtable to storage the elements in the vector are sorted and written out to a file in L0. 
- A prefix-hash memtable allows efficient processing of gets, puts and scans-within-a-key-prefix.


### Memtable Pipelining

RocksDB supports configuring an arbitrary number of memtables for a database. When a memtable is full, it becomes an immutable memtable and a background thread starts flushing its contents to storage. Meanwhile, new writes continue to accumulate to a newly allocated memtable. If the newly allocated memtable is filled up to its limit, it is also converted to an immutable memtable and is inserted into the flush pipeline. The background thread continues to flush all the pipelined immutable memtables to storage. This pipelining increases write throughput of RocksDB, especially when it is operating on slow storage devices.

### Garbage Collection during Memtable Flush

When a memtable is being flushed to storage, an inline-compaction process is executed. Garbages are removed in the same way as compactions. Duplicate updates for the same key are removed from the output stream. Similarly, if an earlier put is hidden by a later delete, then the put is not written to the output file at all. This feature reduces the size of data on storage and write amplification greatly, for some workloads.

## Merge Operator

RocksDB natively supports three types of records, a Put record, a Delete record and a Merge record. When a compaction process encounters a Merge record, it invokes an application-specified method called the Merge Operator. The Merge can combine multiple Put and Merge records into a single one. This powerful feature allows applications that typically do read-modify-writes to avoid the reads altogether. It allows an application to record the intent-of-the-operation as a Merge Record, and the RocksDB compaction process lazily applies that intent to the original value. This feature is described in detail in [Merge Operator](https://github.com/facebook/rocksdb/wiki/Merge-Operator).


