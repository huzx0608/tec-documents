# Block Cache about Rocksdb

Block cache is used for Compressed and Uncompressed Data.
RocksDB uses a LRU [cache for blocks](https://github.com/facebook/rocksdb/wiki/Block-Cache) to serve reads. The block cache is partitioned into two individual caches: the first caches uncompressed blocks and the second caches compressed blocks in RAM. If a compressed block cache is configured, users may wish to enable direct I/O to prevent redundant caching of the same data in OS page cache.


# File Cache about Rocksdb

The Table Cache is a construct that caches open file descriptors. These file descriptors are for sstfiles. An application can specify the maximum size of the Table Cache, or configure RocksDB to always keep all files open, to achieve better performance.

