# Oozie

| |Index|
|---|---|
|1|[Metadata](#metadata)|
|2|[Concepts](#concept)|
|3|[Commands](#command)|

Oozie is a workflow scheduler system to manage Apache Hadoop jobs.

Oozie Workflow jobs are Directed Acyclical Graphs (DAGs) of actions.

Oozie Coordinator jobs are recurrent Oozie Workflow jobs triggered by time (frequency) and data availability.

Oozie is integrated with the rest of the Hadoop stack supporting several types of Hadoop jobs out of the box (such as Java map-reduce, Streaming map-reduce, Pig, Hive, Sqoop and Distcp) as well as system specific jobs (such as Java programs and shell scripts).

Oozie is a scalable, reliable and extensible system.

![Oozie Architecture](https://github.com/barneywill/hadoop_suite/blob/main/imgs/oozie_architecture.jpg)

## 1 <a id='metadata'></a>Metadata
- wf_jobs: workflow instance
- wf_actions: workflow task instance
- coord_jobs: scheduler instance
- coord_actions: scheduler task instance

![Oozie DB](https://github.com/barneywill/hadoop_suite/blob/main/imgs/oozie_db.jpg)

## 2 <a id='concept'></a>Concepts
- Control Node: start、end、kill、decision、fork/join
- Action Node: HDFS、MapReduce、Java、Shell、SSH、Pig、Hive、E-Mail、Sub-Workflow、Sqoop、Distcp
- Workflow: Control Nodes + Action Nodes
- Coordinator: cron

## 3 <a id='command'></a>Commands
```
# show all jobs
$ oozie jobs -oozie http://oozie_server:11000/oozie

# add to environment variables to skip -oozie parameter
$ export OOZIE_URL=http://oozie_server:11000/oozie

# show definition of one job
$ oozie job -definition 0000003-190315230448327-oozie-oozi-W

# show config of one job
$ oozie job -configcontent 0000003-190315230448327-oozie-oozi-W

# show info of one job
$ oozie job -info 0000003-190315230448327-oozie-oozi-W

# show log of one job
$ oozie job -log 0000003-190315230448327-oozie-oozi-W
```

### 3.1 workflow xml
test_sh_wf.xml
```
<workflow-app name="test_sh_wf" xmlns="uri:oozie:workflow:0.5">
    <start to="test_sh_action"/>
    <kill name="Kill">
        <message>Action failed, error message[${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <action name="test_sh_action">
        <shell xmlns="uri:oozie:shell-action:0.1">
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <exec>${script_file}</exec>
            <file>${script_file}</file>
            <capture-output/>
        </shell>
        <ok to="End"/>
        <error to="Kill"/>
    </action>
    <end name="End"/>
</workflow-app>
```

### 3.2 validate and upload to hdfs
```
$ oozie validate test_sh.xml

$ hdfs dfs -put test_sh.xml /user/oozie/wf/
```

### 3.3 job properties
test_sh.properties
```
nameNode=hdfs://namenode:9000
jobTracker=resourcemanager:8032
oozie.wf.application.path=/user/oozie/wf/test_sh.xml
```

### 3.4 start a job
```
$ oozie job -config test_sh.properties -run

$ oozie job -Doozie.wf.rerun.failnodes=true -rerun 0000000-190329232917986-oozie-oozi-W
```
