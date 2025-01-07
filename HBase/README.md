# HBase
Apache HBase™ is the Hadoop database, a distributed, scalable, big data store.

![HBase Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hbase_architecture.jpg)

## 1 Roles

### 1.1 HMaster
the "master server" for HBase

### 1.2 RegionServer
Each RegionServer has:
- One HLog(WAL)
- Many HRegions
  - Many HStore: each corresponds to a column family
    - One MemStore
    - One to Many StoreFile: HFile(SSTable)

## 2 Concepts

### 2.1 LSM Tree (Log-Structured Merge Tree)
SST: Sorting String Table

![LMS Tree](https://github.com/barneywill/hadoop_suite/blob/main/imgs/lsmtree.jpg)

### 2.2 WAL(Write Ahead Log)
A write ahead log is an append-only auxiliary disk-resident structure used for crash and transaction recovery. The changes are first recorded in the log, which must be written to stable storage, before the changes are written to the database. 
It is a standard method for ensuring data integrity. 
- Low-Water Mark: An index in the write ahead log showing which portion of the log can be discarded.
- High-Water Mark: An index in the write ahead log showing the last successful replication.

### 2.3 Namespace, Table, Row, Rowkey, Column Family, Column
- Namespace: a group of tables, like database
- Table: a collection of rows
- Row: rowkey + a collection of column family
- Rowkey: the id of a row, all rows in a table is to be sorted by rowkey
- Column Family: a collection of columns
- Column: a collection of key-value pairs

## 3 Compaction
Because amount of disk-resident table keeps growing, data for the key located in several files, multiple versions of the same record, redundant records that got shadowed by deletes), and the reads will get more and more expensive over the time. 

### 3.1 Minor
on new files being written

### 3.2 Major
The output of a major compaction is one file for one store (one column family inside one region)

## 4 Optimization

### 4.1 Compression
snappy

### 4.2 Block Encoding
FAST_DIFF

### 4.2 Compaction Queue
- offpeak/throughput/merge small store
- hbase.regionserver.global.memstore.size
- hbase.hregion.memstore.flush.size
- hbase.hstore.compaction.max
- hbase.hregion.majorcompaction

