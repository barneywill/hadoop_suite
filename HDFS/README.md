
# HDFS
A distributed file system that provides high-throughput access to application data.

![HDFS Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hdfs_architecture.jpg)

## 1 Roles
### 1.1 NameNode
The namenode manages the filesystem namespace. It maintains the filesystem tree and the metadata for all the files and directories in the tree. fsimage、edit log
- HA
  - Secondary NameNode
  - ZKFC: Zookeeper Failover Controller
    - JournalNode

#### Estimate NameNode Memory
Total = 198 ∗ num(Directory + Files) + 176 ∗ num(blocks) + 2% ∗ size(JVM Memory Size)

### 1.2 DataNode
Datanodes store the actual data.

## 2 File, Block, Replica

![HDFS Files](https://github.com/barneywill/hadoop_suite/blob/main/imgs/hdfs_files.jpg)

## 3 Run

```
bin/hadoop namenode -format
sbin/start-dfs.sh
hdfs dfsadmin -report
```

### URL
```
http://$namenode:9870/dfshealth.html#tab-overview
```

## 4 Commands
```
hdfs dfs -ls /
hdfs dfs -du -h /
hdfs dfsadmin -report
hdfs haadmin -getAllServiceState
hdfs fsck /tmp/36c8b0727c814dc2ae5d520a45fadeb0.51c9f1f1e3652991ed3071c6c0bb0264 -files -blocks -locations -racks
hdfs dfs -setrep 2 /user/hive/warehouse/temp.db/test_ext_o/000000_0
hdfs balancer -threshold 10
hadoop distcp hdfs://master1:8020/foo/a hdfs://master1:8020/foo/b hdfs://master2:8020/bar/foo
hdfs dfsadmin -safemode leave
```