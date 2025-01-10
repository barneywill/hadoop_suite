# Hive Performance Tuning

| |Index|
|---|---|
|1|[Compression(MR)](#compression)|
|2|[Decompose(Partition, Bucket, Skewed)](#decompose)|
|3|[Index](#index)|
|4|[File Format(ORC, Parquet, Snappy)](#format)|
|5|[Vectorization](#vectorization)|
|6|[Join(Map Join, Bucket Map Join, SMB Join, Skew Join)](#join)|
|7|[Merge small files](#merge)|
|8|[CBO(Cost-Based Optimizer)](#cbo)|
|9|[Correlation](#correlation)|
|10|[Write a good SQL](#sql)|
|11|[Engine](#engine)|
|12|[Other](#other)|

## 1 <a id='compression'></a>Compression

### 1.1 MR, Compression
Default engine is mr.
```
hive> set hive.execution.engine;
hive.execution.engine=mr
```

#### 1.1.1 Turn on mapreduce.map.output.compress
mapred-site.xml
```
    <property>
        <name>mapreduce.map.output.compress</name>  
        <value>true</value>
    </property>
    <property>
        <name>mapred.map.output.compress.codec</name>  
        <value>org.apache.hadoop.io.compress.SnappyCodec</value>
    </property>
```

#### 1.1.2 Set multiple local dirs, better distributed on different disks
yarn-site.xml
```
    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>${hadoop.tmp.dir}/nm-local-dir</value>
    </property>
```

### 1.2 Turn on hive.exec.compress.output
This controls whether the final outputs of a query (to a local/hdfs file or a Hive table) is compressed. The compression codec and other options are determined from Hadoop configuration variables mapred.output.compress* .
```
set hive.exec.compress.output=true;
```

### 1.3 Turn on hive.exec.compress.intermediate
This controls whether intermediate files produced by Hive between multiple map-reduce jobs are compressed. The compression codec and other options are determined from Hadoop configuration variables mapred.output.compress*.
```
set hive.exec.compress.intermediate=true;
```

## 2 <a id='decompose'></a>Decompose
Decompose table data sets into more manageable parts.

### 2.1 Partition
Hive Partitioning provides a way of segregating hive table data into multiple files/directories.

Partitioned tables can be created using the PARTITIONED BY clause. A table can have one or more partition columns and a separate data directory is created for each distinct value combination in the partition columns. Further, tables or partitions can be bucketed using CLUSTERED BY columns, and data can be sorted within that bucket via SORT BY columns. This can improve performance on certain kinds of queries.
```
PARTITIONED BY(dt STRING, country STRING)
```

### 2.2 Bucket
Bucketed tables are fantastic in that they allow much more efficient sampling than do non-bucketed tables, and they may later allow for time saving operations such as mapside joins. However, the bucketing specified at table creation is not enforced when the table is written to, and so it is possible for the table's metadata to advertise properties which are not upheld by the table's actual layout. This should obviously be avoided.

How does Hive distribute the rows across the buckets? In general, the bucket number is determined by the expression hash_function(bucketing_column) mod num_buckets.

Generally, in the table directory, each bucket is just a file, and Bucket numbering is 1-based.
```
CLUSTERED BY(userid) SORTED BY(viewTime) INTO 32 BUCKETS
```

### 2.3 Skewed
This feature can be used to improve performance for tables where one or more columns have skewed values. By specifying the values that appear very often (heavy skew) Hive will split those out into separate files (or directories in case of list bucketing) automatically and take this fact into account during queries so that it can skip or include the whole file (or directory in case of list bucketing) if possible.
```
SKEWED BY (key) ON (1,5,6) [STORED AS DIRECTORIES]
```

## 3 <a id='index'></a>Index
Indexing Is Removed since 3.0. 
There are alternate options which might work similarily to indexing:
- Materialized views with automatic rewriting can result in very similar results. Hive 2.3.0 adds support for materialzed views.
- Using columnar file formats (Parquet, ORC) – they can do selective scanning; they may even skip entire files/blocks.

### 3.1 Materialized view
Traditionally, one of the most powerful techniques used to accelerate query processing in data warehouses is the pre-computation of relevant summaries or materialized views.

Using a materialized view, the optimizer can compare old and new tables, rewrite queries to accelerate processing, and manage maintenance of the materialized view when data updates occur. The optimizer can use a materialized view to fully or partially rewrite projections, filters, joins, and aggregations. Hive stores materialized views in the Hive warehouse or Druid.
```
CREATE MATERIALIZED VIEW [IF NOT EXISTS] [db_name.]materialized_view_name AS <query>;
```

## 4 <a id='format'></a>File Format

### 4.1 Text + lzo
```
STORED AS INPUTFORMAT
'com.hadoop.mapred.DeprecatedLzoTextInputFormat'
OUTPUTFORMAT
'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
```

### 4.2 ORC + Snappy
```
STORED AS ORC
STORED AS ORC TBLPROPERTIES ("orc.compress"="SNAPPY")
```

#### 4.2.1 Zerocopy
ORC can use the new HDFS Caching APIs and the ZeroCopy readers to avoid extra data copies into memory while scanning files.
```
set hive.orc.zerocopy=true;
```

### 4.3 Parquet + Snappy
```
STORED AS PARQUET
TBLPROPERTIES ("parquet.compression"="SNAPPY")
```

## 5 <a id='vectorization'></a>Vectorization
Vectorized query execution is a Hive feature that greatly reduces the CPU usage for typical query operations like scans, filters, aggregates, and joins. A standard query execution system processes one row at a time. This involves long code paths and significant metadata interpretation in the inner loop of execution. Vectorized query execution streamlines operations by processing a block of 1024 rows at a time. Within the block, each column is stored as a vector (an array of a primitive data type). Simple operations like arithmetic and comparisons are done by quickly iterating through the vectors in a tight loop, with no or very few function calls or conditional branches inside the loop. These loops compile in a streamlined way that uses relatively few instructions and finishes each instruction in fewer clock cycles, on average, by effectively using the processor pipeline and cache memory.
```
set hive.vectorized.execution.enabled=true;
set hive.vectorized.execution.reduce.enabled=true;
set hive.vectorized.execution.reduce.groupby.enabled=true;
```

## 6 <a id='join'></a>Join

### 6.1 Common Join
Use Mappers to do the parallel sort of the tables on the join keys, which are then passed on to reducers. All of the tuples with same key is given to same reducer. A reducer may get tuples for more than one key. Key for tuple will also include table id, thus sorted output from two different tables with same key can be recognized. Reducers will merge the sorted stream to get join output.

### 6.2 Map Join
Useful for star schema joins, this joining algorithm keeps all of the small tables (dimension tables) in memory in all of the mappers and big table (fact table) is streamed over it in the mapper. This avoids shuffling cost that is inherent in Common-Join. For each of the small table (dimension table) a hash table would be created using join key as the hash table key.
```
set hive.auto.convert.join=true;
set hive.auto.convert.join.noconditionaltask = true;
set hive.auto.convert.join.noconditionaltask.size = 10000000;
```

MAPJOINs are processed by loading the smaller table into an in-memory hash map and matching keys with the larger table as they are streamed through. The prior implementation has this division of labor:
- Local work:
  - read records via standard table scan (including filters and projections) from source on local machine
  - build hashtable in memory
  - write hashtable to local disk
  - upload hashtable to dfs
  - add hashtable to distributed cache
- Map task
  - read hashtable from local disk (distributed cache) into memory
  - match records' keys against hashtable
  - combine matches and write to output
- No reduce task

### 6.3 Bucket Map Join
If the joining keys of map-join are bucketed then instead of keeping whole of small table (dimension table) in every mapper, only the matching buckets will be kept. This reduces the memory footprint of the map-join.
```
set hive.auto.convert.sortmerge.join=true;
set hive.optimize.bucketmapjoin = true;
```

### 6.4 SMB Join (Sort Merge Bucket)
This is an optimization on Bucket Map Join; if data to be joined is already sorted on joining keys then hash table creation is avoided and instead a sort merge join algorithm is used.
```
set hive.optimize.bucketmapjoin.sortedmerge = true;
```
SMB joins are used wherever the tables are sorted and bucketed. The join boils down to just merging the already sorted tables, allowing this operation to be faster than an ordinary map-join. However, if the tables are partitioned, there could be a slow down as each mapper would need to get a very small chunk of a partition which has a single key.

### 6.5 Skew Join
If the distribution of data is skewed for some specific values, then join performance may suffer since some of the instances of join operators (reducers in map-reduce world) may get over loaded and others may get under utilized. On user hint, hive would rewrite a join query around skew value as union of joins.
```
set hive.optimize.skewjoin=true;
set hive.skewjoin.key=100000;
```
Whether to enable skew join optimization. The algorithm is as follows: At runtime, detect the keys with a large skew. Instead of processing those keys, store them temporarily in an HDFS directory. In a follow-up map-reduce job, process those skewed keys. The same key need not be skewed for all the tables, and so, the follow-up map-reduce job (for the skewed keys) would be much faster, since it would be a map-join.

### 6.6 Semi Join
LEFT SEMI JOIN implements the uncorrelated IN/EXISTS subquery semantics in an efficient way. As of Hive 0.13 the IN/NOT IN/EXISTS/NOT EXISTS operators are supported using subqueries so most of these JOINs don't have to be performed manually anymore.

## 7 <a id='merge'></a>Merge small files
Whether to combine small input files so that fewer mappers are spawned.
```
set hive.hadoop.supports.splittable.combineinputformat=true;
```

The minimum size chunk that map input should be split into. Note that some file formats may have minimum split sizes that take priority over this setting.
```
set mapreduce.input.fileinputformat.split.minsize=1;
set mapreduce.input.fileinputformat.split.maxsize=256000000;
```

Merge small files at the end of a map-only job.
```
set hive.merge.mapfiles=true;
```

Merge small files at the end of a map-reduce job.
```
set hive.merge.mapredfiles=true;
```

## 8 <a id='cbo'></a>Stats, CBO(Cost-Based Optimizer)
Most of the existing query optimizations in Hive are about minimizing shuffling cost. Currently user would have to submit an optimized query to Hive with right join order for query to be executed efficiently. Logical optimizations in Hive are limited to filter push down, projection pruning and partition pruning. Cost based logical optimizations can significantly improve Apache Hive’s query latency and ease of use.

Join reordering and join algorithm selection are few of the optimizations that can benefit from a cost based optimizer. Cost based optimizer would free up user from having to rearrange joins in the right order or from having to specify join algorithm by using query hints and configuration options. This can potentially free up users to model their reporting and ETL needs close to business process without having to worry about query optimizations.

Calcite is an open source cost based query optimizer and query execution framework. Calcite currently has more than fifty query optimization rules that can rewrite query tree, and an efficient plan pruner that can select cheapest query plan in an optimal manner.

CBO will be introduced in to Hive in a Phased manner. In the first phase, Calcite would be used to reorder joins and to pick right join algorithm so as to reduce query latency. Table cardinality and Boundary statistics will be used for this cost based optimizations.


Hive’s Cost-Based Optimizer (CBO) is a core component in Hive’s query processing engine. Powered by Apache Calcite, the CBO optimizes and calculates the cost of various plans for a query.

The main goal of a CBO is to generate efficient execution plans by examining the tables and conditions specified in the query, ultimately cutting down on query execution time and reducing resource utilization. After parsing, a query gets converted to a logical tree (Abstract Syntax Tree) that represents the operations that the query must perform, such as reading a particular table or performing an inner JOIN.

Calcite applies various optimizations such as query rewrite, JOIN reordering, and deriving implied predicates and JOIN elimination to produce logically equivalent plans. The current model prefers bushy plans for maximum parallelism. Each logical plan is assigned a cost based in number of distinct value based heuristics.

Calcite has an efficient plan pruner that can select the cheapest query plan. The chosen logical plan is then converted by Hive to a physical operator tree, optimized and converted to Tez jobs, and then executed on the Hadoop cluster.

Enabling Cost-Based Optimization
```
set hive.cbo.enable=true;
set hive.stats.autogather=true;
set hive.compute.query.using.stats=true;
set hive.stats.fetch.partition.stats=true;
set hive.stats.fetch.column.stats=true;
```

Generating Hive Statistics
```
ANALYZE TABLE [table_name] COMPUTE STATISTICS;
ANALYZE TABLE [table_name] PARTITION(partition_column) COMPUTE STATISTICS;
ANALYZE TABLE [table_name] COMPUTE STATISTICS for COLUMNS [comma_separated_column_list];
```

Viewing Generated Statistics
```
DESCRIBE [EXTENDED] table_name;
DESCRIBE FORMATTED [db_name.]table_name.column_name;
```

## 9 <a id='correlation'></a>Correlation
In Hadoop environments, an SQL query submitted to Hive will be evaluated in distributed systems. Thus, after generating a query operator tree representing the submitted SQL query, Hive needs to determine what operations can be executed in a task which will be evalauted in a single node. Also, since a MapReduce job can shuffle data data once, Hive also needs to cut the tree to multiple MapReduce jobs. It is important to cut an operator tree to multiple MapReduce in a good way, so the generated plan can evaluate the query efficiently.

In a more complex query, correlation-unaware query planning can generate a very inefficient execution plan and result in poor performance.
```
set hive.optimize.correlation=true;
```

## 10 <a id='sql'></a>Write a good SQL

### 10.1 execution plan
```
hive> explain $sql;
```

### 10.2 Good practice

## 11 <a id='engine'></a>Engine
```
set hive.query.engine=spark;
```

## 12 <a id='other'></a>Other

### 12.1 Parallel
Whether to execute jobs in parallel. Applies to MapReduce jobs that can run in parallel, for example jobs processing different source tables before a join. As of Hive 0.14, also applies to move tasks that can run in parallel, for example moving files to insert targets during multi-insert.
```
set hive.exec.parallel=true;
```

### 12.2 Sample
Whether to enable to optimization to trying a smaller subset of data for simple LIMIT first.
```
set hive.limit.optimize.enable=true;
```

Uses sampling on order-by clause for parallel execution.
```
set hive.optimize.sampling.orderby=true;
```
