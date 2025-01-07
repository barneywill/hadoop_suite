# YARN
A framework for job scheduling and cluster resource management.

Moving Computation is Cheaper than Moving Data

![YARN Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/yarn_architecture.jpg)

## 1 Roles

### 1.1 ResourceManager
The ResourceManager is the ultimate authority that arbitrates resources among all the applications in the system.

### 1.2 NodeManager
The NodeManager is the per-machine framework agent who is responsible for containers, monitoring their resource usage (cpu, memory, disk, network) and reporting the same to the ResourceManager/Scheduler.

## 2 Scheduler
- FIFO
- Capacity
- Fair

## 3 Run

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

## 4 Commands

```
yarn node
yarn queue -status root
yarn top
yarn application -list -appStates ALL -appTypes MapReduce|more
yarn rmadmin
yarn logs -applicationid=$applicationId
```

