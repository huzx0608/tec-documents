# Compacation about Rocksdb



## Compaction Status



```
* Level：也就是 LSM 的 level，Sum 表示的是所有 level 的总和，而 Int 则表示从上一次 stats 到现在的数据。
* Files：有两个值 a/b，a 用来表示当前 level 有多少文件，而 b 则用来表示当前用多少个线程正在对这层 level 做 compaction。
* Size(MB}：当前 level 总共多大，用 MB 表示。
* Score：除开 level 0，其他几层的 score 都是通过 (current level size) / (max level size) 来计算，通常 0 和 1 是正常的，但如果大于 1 了，表明这层 level 就需要做 compaction 了。对于 level 0，我们通过 (current number of files) / (file number compaction trigger) 来计算。
* Read(GB}：对 level N 和 N + 1 做 compaction 的时候总的读取数据量
* Rn(GB}：对 level N 和 N + 1 做 compaction 的时候从 level N 读取的数据量
* Rnp1(GB}：对 level N 和 N + 1 做 compaction 的时候从 level N + 1读取的数据量
* Write(GB}：对 level N 和 N + 1 做 compaction 的时候总的写入数据量
* Wnew(GB}：新写入 level N + 1 的数据量，使用 (total bytes written to N+1) - (bytes read from N+1 during compaction with level N)
* Moved(GB}：直接移动到 level N + 1 的数据量。这里仅仅只会更新 manifest，并没有其他 IO 操作，表明之前在 level X 的文件现在在 level Y 了。
* W-Amp：从 level N 到 N + 1 的写放大，使用 (total bytes written to level N+1) / (total bytes read from level N) 计算。
* Rd(MB/s}：在 level N 到 N + 1 做 compaction 的时候读取速度。使用 Read(GB) * 1024 / duration 计算，duration 就是从 N 到 N + 1 compaction 的时间。
* Wr(MB/s}：在 level N 到 N + 1 做 compaction 的时候写入速度，类似 Rd(MB/s)。
* Comp(sec}：在 level N 到 N + 1 做 compaction 的总的时间。
* Comp(cnt}：在 level N 到 N + 1 做 compaction 的总的次数。
* Avg(sec}：在 level N 到 N + 1 做 compaction 平均时间。
* KeyIn：在 level N 到 N + 1 做 compaction 的时候比较的 record 的数量
* KeyDrop：在 level N 到 N + 1 做 compaction 的时候直接丢弃的 record 的数量
```



## Compaction Status



```dart
Uptime(secs): 727897.9 total, 727897.9 interval
Flush(GB): cumulative 0.778（累积 flush数据量）, interval 0.296（间隔 flush数据量）
AddFile(GB): cumulative 0.000（累积fastload导入数据量 ）, interval 0.000（间隔fastload导入数据量）
AddFile(Total Files): cumulative 0（累积Fastload导入文件个数）, interval 0（(间隔Fastload导入文件个数）
AddFile(L0 Files): cumulative 0(累积Fastload导入L0文件个数), interval 0(间隔Fastload导入L0文件个数)
AddFile(Keys): cumulative 0(累积Fastload导入key数), interval 0(间隔Fastload导入key数)
Cumulative compaction: 1.42 GB write(compact写数据量，包括memtable Flush到L0), 0.00 MB/s write, 0.81 GB read(compact读文件数据量，Memtable Flush到L0没有读数据量),0.00 MB/s read, 69.6 seconds
Interval compaction: 0.94 GB write, 0.00 MB/s write, 0.81 GB read, 0.00 MB/s read, 45.7 seconds
Stalls(count): 0 level0_slowdown, 0 level0_slowdown_with_compaction, 0 level0_numfiles, 0 level0_numfiles_with_compaction, 0 stop for pending_compaction_bytes, 0 slowdown for pending_co
```