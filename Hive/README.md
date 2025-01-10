# Hive
A data warehouse infrastructure that provides data summarization and ad hoc querying.

![Hive Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hive_architect.jpg)

## 1 Roles

### 1.1 Metastore
Store the metadata of objects such as their location, schema details also contain the detail about the cluster partition to keep a track of dataset progress.

![Hive Matastore](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hive_metastore.jpg)

#### metadata
- VERSION
- DBS
- TBLS
- PARTITIONS
- COLUMNS_V2
- SERDES
- FUNCS
- IDXS
- TAB_COL_STATS

### 1.2 HiveServer2
Provide an interaction of external clients with Hive using JDBC or ODBC protocol.

## 2 Clients

```
# hive
/bin/hive

# beeline
bin/beeline -u jdbc:hive2://$HS2_HOST:$HS2_PORT
```

## 3 SERDE
Serialize/Deserilize
- csv
  - org.apache.hadoop.hive.serde2.OpenCSVSerde
- json
  - org.apache.hive.hcatalog.data.JsonSerDe
- xml
  - com.ibm.spss.hive.serde2.xml.XmlSerDe
- parquet
  - org.apache.hadoop.hive.ql.io.parquet.serde.ParquetHiveSerDe
- orc
  - org.apache.hadoop.hive.ql.io.orc.OrcSerde
- hbase
  - org.apache.hadoop.hive.hbase.HBaseSerDe

### Compression
- lzo
- snappy

## 4 External Table
- location
- integration: hbase, es

## 5 Join
- map join、broadcast
  - hive.auto.convert.join
- bucket map join
  - hive.optimize.bucketmapjoin
- sorted merge bucket join
  - hive.optimize.bucketmapjoin.sortedmerge
- skew join
  - hive.optimize.skewjoin
- left semi join

## 6 Execution
SQL -> AST(Abstract Syntax Tree) -> Task(MapRedTask，FetchTask) -> QueryPlan(Tasks) -> Job(Yarn)

- Map
  - TableScan, Filter Operator, Selector Operator
- Reduce
  - Group By Operator, File Output Operator

![Hive Execution](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hive_execution.jpg)

### CBO(Cost Based Optimize)
The main goal of a CBO is to generate efficient execution plans by examining the tables and conditions specified in the query, ultimately cutting down on query execution time and reducing resource utilization.

## 7 <a id='python' href='https://github.com/barneywill/hadoop_suite/tree/main/Hive/hive_performance_tuning.md'>Performance Tuning</a>
