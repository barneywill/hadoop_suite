# Benchmark

| |Index|
|---|---|
|1|[HiBench](#hibench)|
|2|[TPC-DS(hive-testbench)](#tpc)|
|3|[SSB(Star Schema Benchmark)](#ssb)|

## 1 <a id='hibench'></a>HiBench
https://github.com/intel-hadoop/HiBench

HiBench is a big data benchmark suite that helps evaluate different big data frameworks in terms of speed, throughput and system resource utilizations. It contains a set of Hadoop, Spark and streaming workloads, including Sort, WordCount, TeraSort, Sleep, SQL, PageRank, Nutch indexing, Bayes, Kmeans, NWeight and enhanced DFSIO, etc. It also contains several streaming workloads for Spark Streaming, Flink, Storm and Gearpump.

There are totally 19 workloads in HiBench.

### 1.1 Build
```
# download
$ wget https://github.com/intel-hadoop/HiBench/archive/HiBench-7.0.tar.gz
$ tar xvf HiBench-7.0.tar.gz
$ cd HiBench-HiBench-7.0

# build all
$ mvn -Dspark=2.1 -Dscala=2.11 clean package
# build hadoopbench and sparkbench
$ mvn -Phadoopbench -Psparkbench -Dspark=2.1 -Dscala=2.11 clean package

# prepare
$ cp conf/hadoop.conf.template conf/hadoop.conf
$ vi conf/hadoop.conf

$ cp conf/spark.conf.template conf/spark.conf
$ vi conf/spark.conf

$ vi conf/hibench.conf
# Data scale profile. Available value is tiny, small, large, huge, gigantic and bigdata.
# The definition of these profiles can be found in the workload's conf file i.e. conf/workloads/micro/wordcount.conf
hibench.scale.profile bigdata
```

### 1.2 Run
scan/aggregation/join
```
$ bin/workloads/sql/scan/prepare/prepare.sh
$ bin/workloads/sql/scan/spark/run.sh

$ bin/workloads/sql/join/prepare/prepare.sh
$ bin/workloads/sql/join/spark/run.sh

$ bin/workloads/sql/aggregation/prepare/prepare.sh
$ bin/workloads/sql/aggregation/spark/run.sh
```

## 2 <a id='tpc'></a>TPC
http://www.tpc.org/

The TPC is a non-profit corporation founded to define transaction processing and database benchmarks and to disseminate objective, verifiable TPC performance data to the industry.

The term transaction is often applied to a wide variety of business and computer functions. Looked at as a computer function, a transaction could refer to a set of operations including disk read/writes, operating system calls, or some form of data transfer from one subsystem to another.

While TPC benchmarks certainly involve the measurement and evaluation of computer functions and operations, the TPC regards a transaction as it is commonly understood in the business world: a commercial exchange of goods, services, or money. A typical transaction, as defined by the TPC, would include the updating to a database system for such things as inventory control (goods), airline reservations (services), or banking (money).

In these environments, a number of customers or service representatives input and manage their transactions via a terminal or desktop computer connected to a database. Typically, the TPC produces benchmarks that measure transaction processing (TP) and database (DB) performance in terms of how many transactions a given system and database can perform per unit of time, e.g., transactions per second or transactions per minute.

![TPC](https://github.com/barneywill/hadoop_suite/blob/main/imgs/tpc.jpg)

### 2.1 TPC-DS
http://www.tpc.org/tpcds/default.asp

http://www.tpc.org/tpc_documents_current_versions/pdf/tpc-ds_v2.10.1.pdf

A simple schema for decision support systems or data warehouses is the star schema, where events are collected in large fact tables, while smaller supporting tables (dimensions) are used to describe the data.

The TPC-DS is an example of such a schema. It models a typical retail warehouse where the events are sales and typical dimensions are date of sale, time of sale, or demographic of the purchasing party. 

The TPC Benchmark DS (TPC-DS) is a decision support benchmark that models several generally applicable aspects of a decision support system, including queries and data maintenance. The benchmark provides a representative evaluation of performance as a general purpose decision support system. A benchmark result measures query response time in single user mode, query throughput in multi user mode and data maintenance performance for a given hardware, operating system, and data processing system configuration under a controlled, complex, multi-user decision support workload. The purpose of TPC benchmarks is to provide relevant, objective performance data to industry users. TPC-DS Version 2 enables emerging technologies, such as Big Data systems, to execute the benchmark.

#### 2.1.1 Build
```
http://www.tpc.org/tpc_documents_current_versions/current_specifications.asp

$ unzip TPC-DS_Tools_v2.10.1.zip
$ cd v2.10.1rc3/tools
$ make

# create table statements
tpcds.sql
tpcds_ri.sql
tpcds_source.sql
```

#### 2.1.2 Generate data and query sql
```
# generate data
$ mkdir /tmp/tpcdsdata
$ ./dsdgen -SCALE 1GB -DIR /tmp/tpcdsdata -parallel 4 -child 4

# generate query sql
$ ./dsqgen -input ../query_templates/templates.lst -directory ../query_templates -dialect oracle -scale 1GB -OUTPUT_DIR /tmp/tpcdsdata/query_oracle
```

#### 2.1.3 hive-testbench
https://github.com/hortonworks/hive-testbench

##### 2.1.3.1 Build
```
$ wget https://github.com/hortonworks/hive-testbench/archive/hive14.zip
$ unzip hive14.zip
$ cd hive-testbench-hive14/
$ ./tpcds-build.sh
```

##### 2.1.3.2 Generate data and query sql
```
# generate
$ export FORMAT=parquet
$ ./tpcds-setup.sh 1000
```

##### 2.1.3.3 Run query
```
$ cd sample-queries-tpcds
hive> use tpcds_bin_partitioned_parquet_10;
hive> source query12.sql;

# beeline to hiveserver2
$ perl runSuite.pl tpcds 10 parquet "$HIVE_HOME/bin/beeline -u jdbc:hive2://localhost:10000"
# beeline to spark thrift server
$ perl runSuite.pl tpcds 10 parquet "$SPARK_HOME/bin/beeline -u jdbc:hive2://localhost:11111"
# beeline to impala
perl runSuite.pl tpcds 10 parquet "$HIVE_HOME/bin/beeline -d com.cloudera.impala.jdbc4.Driver -u jdbc:impala://localhost:21050"
```

![Hive testbench](https://github.com/barneywill/hadoop_suite/blob/main/imgs/testbench.jpg)

## 3 <a id='ssb'></a>SSB(Star Schema Benchmark)
https://www.cs.umb.edu/~poneil/StarSchemaB.PDF

Star Schema Benchmark(SSB) is a lightweight performance test set in the data warehouse scenario. SSB provides a simplified star schema data based on TPC-H, which is mainly used to test the performance of multi-table JOIN query under star schema. In addition, the industry usually flattens SSB into a wide table model (Referred as: SSB flat) to test the performance of the query engine, refer to Clickhouse.

### 3.1 Generate Data
https://github.com/vadimtk/ssb-dbgen
```
./dbgen -s 100 -T a
```

### 3.2 Run query
```
./qgen -s 100
```


