# Hive
A data warehouse infrastructure that provides data summarization and ad hoc querying.

| |Index|
|---|---|
|1|[Roles(Metastore, HiveServer2)](#role)|
|2|[Clients](#client)|
|3|[SERDE](#serde)|
|4|[External Table](#external)|
|5|[Join](#join)|
|6|[Execution](#execution)|
|7|[Performance Tuning](#tuning)|

![Hive Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hive_architect.jpg)

## 1 <a id='role'></a>Roles

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

## 2 <a id='client'></a>Clients

```
# hive
/bin/hive

# beeline
bin/beeline -u jdbc:hive2://$HS2_HOST:$HS2_PORT
```

## 3 <a id='serde'></a>SERDE
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

## 4 <a id='external'></a>External Table
- location
- integration: hbase, es

## 5 <a id='join'></a>Join
- map join、broadcast
  - hive.auto.convert.join
- bucket map join
  - hive.optimize.bucketmapjoin
- sorted merge bucket join
  - hive.optimize.bucketmapjoin.sortedmerge
- skew join
  - hive.optimize.skewjoin
- left semi join

## 6 <a id='execution'></a>Execution
SQL -> AST(Abstract Syntax Tree) -> Task(MapRedTask，FetchTask) -> QueryPlan(Tasks) -> Job(Yarn)

- Map
  - TableScan, Filter Operator, Selector Operator
- Reduce
  - Group By Operator, File Output Operator

![Hive Execution](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hive_execution.jpg)

### CBO(Cost Based Optimize)
The main goal of a CBO is to generate efficient execution plans by examining the tables and conditions specified in the query, ultimately cutting down on query execution time and reducing resource utilization.

## 7 <a id='tuning' href='https://github.com/barneywill/hadoop_suite/tree/main/Hive/hive_performance_tuning.md'>Performance Tuning</a>
