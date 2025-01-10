# YARN
A framework for job scheduling and cluster resource management.

Moving Computation is Cheaper than Moving Data

| |Index|
|---|---|
|1|[Roles(ResourceManager, NodeManager, HistoryServer)](#role)|
|2|[Scheduler](#scheduler)|
|3|[Run](#run)|
|4|[Commands](#command)|

![YARN Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/yarn_architecture.jpg)

## 1 <a id='role'></a>Roles

### 1.1 ResourceManager
The ResourceManager is the ultimate authority that arbitrates resources among all the applications in the system.

### 1.2 NodeManager
The NodeManager is the per-machine framework agent who is responsible for containers, monitoring their resource usage (cpu, memory, disk, network) and reporting the same to the ResourceManager/Scheduler.

### 1.3 HistoryServer

## 2 <a id='scheduler'></a>Scheduler
- FIFO
- Capacity
- Fair

## 3 <a id='run'></a>Run

```
sbin/start-yarn.sh
yarn node -list
```

### URL

```
http://$ip:8088/ws/v1/cluster/metrics
http://$ip:8088/ws/v1/cluster/scheduler
http://$ip:8088/ws/v1/cluster/apps
```

## 4 <a id='command'></a>Commands

```
yarn node
yarn queue -status root
yarn top
yarn application -list -appStates ALL -appTypes MapReduce|more
yarn rmadmin
yarn logs -applicationid=$applicationId
```

