# Tikv Introduction

## Tikv architecture


RocksDB作为TiKV的核心存储引擎，用于存储Raft日志以及用户数据。每个TiKV实例中有两个RocksDB实例，一个用于存储Raft日志（通常被称为raftdb），另一个用于存储用户数据以及MVCC信息（通常被称为kvdb）。kvdb中有四个ColumnFamily：raft、lock、default和write：

- raft列：用于存储各个Region的元信息，仅占极少量空间
- lock列：用于存储悲观事务的悲观锁以及分布式事务的一阶段Prewrite锁。当用户的事务提交之后， lock cf中对应的数据会很快删除掉，因此大部分情况下lock cf中的数据也很少（少于1GB）。如果lock cf中的数据大量增加，说明有大量事务等待提交，系统出现了bug或者故障。
- write列：用于存储用户真实的写入数据以及MVCC信息（该数据所属事务的开始时间以及提交时间）。当用户写入了一行数据时，如果该行数据长度小于255字节，那么会被存储write列中，否则的话该行数据会被存入到default列中。由于TiDB的非unique 索引存储的value为空，unique索引存储的value为主键索引，因此二级索引只会占用writecf的空间。
- default列：用于存储超过255字节长度的数据

