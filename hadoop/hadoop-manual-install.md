# Hadoop setup: "distributed mode" on single machine
## Documentation
Manual:
- http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html

Other docs:
- https://habrahabr.ru/post/319048/ (rus)
  - https://github.com/hadoopfromscratch/hadoopfromscratch/blob/master/install.sh

## Set up your machine
[Machine setup](/common/setup.md)

## Unpack hadoop archive
Unpack archive, make symlink
```
$ pwd
/home/hadoop

$ sudo -Hu hadoop -g hadoop tar -xf /path/to/hadoop-2.7.4.tar.gz
$ sudo -Hu hadoop -g hadoop ln -sf hadoop-2.7.4 hadoop

$ ls -l /home/hadoop/
lrwxrwxrwx  1 hadoop hadoop   12 сен 18 21:56 hadoop -> hadoop-2.7.4/
drwxr-xr-x 10 hadoop hadoop 4096 авг  1 04:09 hadoop-2.7.4/
```

## Create SSH keys, set up SSH server if need
See [Machine setup](/common/setup.md)

## Edit config files
Set JAVA_HOME var (If you have appropriate ```/etc/environment``` (see [Machine setup](/common/setup.md)), you can skip this step)
http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Prepare_to_Start_the_Hadoop_Cluster
>Unpack the downloaded Hadoop distribution. In the distribution, edit the file ```etc/hadoop/hadoop-env.sh``` to define some parameters as follows:

```etc/hadoop/hadoop-env.sh```:
```
# set to the root of your Java installation
export JAVA_HOME=/usr/java/latest
```
Log dir setup

Modify etc/hadoop/hadoop-env.sh
```
$ cp hadoop-env.sh hadoop-env.sh.orig
# uncomment in etc/hadoop/hadoop-env.sh "export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}/$USER"
#
hadoop@machinename:~/hadoop/etc/hadoop$ diff -U1 hadoop-env.sh.orig hadoop-env.sh
--- hadoop-env.sh.orig	2017-09-24 23:42:54.854334597 +0300
+++ hadoop-env.sh	2017-09-24 23:47:40.214401506 +0300
@@ -71,3 +71,3 @@
 # Where log files are stored.  $HADOOP_HOME/logs by default.
-#export HADOOP_LOG_DIR=${HADOOP_LOG_DIR}/$USER
+export HADOOP_LOG_DIR=/home/hadoop/hadoop-logs

```

Create logs dir:
```
hadoop@machinename:~$ mkdir /home/hadoop/hadoop-logs
```

```etc/hadoop/core-site.xml```:
```
<configuration>
    <property><name>fs.defaultFS</name><value>hdfs://localhost:9000</value></property>
    <property><name>hadoop.tmp.dir</name><value>/home/hadoop/hadoop-data</value></property>
</configuration>
```

```etc/hadoop/hdfs-site.xml```:
```
<configuration>
    <property><name>dfs.replication</name><value>1</value></property>
</configuration>
```

```etc/hadoop/mapred-site.xml```:
```
<configuration>
  <property><name>mapreduce.framework.name</name><value>yarn</value></property>
  <property><name>yarn.app.mapreduce.am.resource.mb</name><value>1512</value></property>
  <property><name>mapreduce.map.memory.mb</name><value>1900</value></property>
  <property><name>mapreduce.reduce.memory.mb</name><value>1900</value></property>
  <property><name>mapreduce.jobhistory.address</name><value>master.local:10020</value></property>
  <property><name>mapreduce.jobhistory.webapp.address</name><value>master.local:19888</value></property>
</configuration>
```

```etc/hadoop/yarn-site.xml:```:
```
<configuration>
  <property><name>yarn.resourcemanager.hostname</name><value>master.local</value></property>
  <property><name>yarn.scheduler.minimum-allocation-mb</name><value>2000</value></property>
  <property><name>yarn.scheduler.maximum-allocation-mb</name><value>2000</value></property>
  <property><name>yarn.scheduler.minimum-allocation-vcores</name><value>1</value></property>
  <property><name>yarn.scheduler.maximum-allocation-vcores</name><value>1</value></property>
  <property><name>yarn.nodemanager.resource.memory-mb</name><value>8192</value></property>
  <property><name>yarn.nodemanager.resource.cpu-vcores</name><value>4</value></property>
  <property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property>
  <property><name>yarn.log-aggregation-enable</name><value>true</value></property>
</configuration>
```

Edit ```etc/hadoop/slaves``` file:
```
echo "master.local" > etc/hadoop/slaves
```

## Create data dir
```
$ pwd
/home/hadoop
$ mkdir hadoop-data
```

## Start
Add bin/ sbin/ files to PATH:
```
echo "export PATH=$PATH:/home/hadoop/hadoop/bin:/home/hadoop/hadoop/sbin" >> ~/.bashrc
# relogin
```

Format node:
```
$ hdfs namenode -format
```
Start services
```
hadoop-daemon.sh start namenode
hadoop-daemon.sh start datanode
yarn-daemon.sh start resourcemanager
yarn-daemon.sh start nodemanager
mr-jobhistory-daemon.sh start historyserver
```

Stop services
```
mr-jobhistory-daemon.sh start historyserver
yarn-daemon.sh start nodemanager
yarn-daemon.sh start resourcemanager
hadoop-daemon.sh start datanode
hadoop-daemon.sh start namenode
```

# Run test application
```
$ pwd
/home/hadoop
$ yarn jar ./hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.4.jar pi 10 16
Number of Maps  = 10
Samples per Map = 16
Wrote input for Map #0
Wrote input for Map #1
Wrote input for Map #2
Wrote input for Map #3
Wrote input for Map #4
Wrote input for Map #5
Wrote input for Map #6
Wrote input for Map #7
Wrote input for Map #8
Wrote input for Map #9
Starting Job
17/09/26 00:35:07 INFO client.RMProxy: Connecting to ResourceManager at master.local/127.0.0.1:8032
17/09/26 00:35:07 INFO input.FileInputFormat: Total input paths to process : 10
17/09/26 00:35:08 INFO mapreduce.JobSubmitter: number of splits:10
17/09/26 00:35:08 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1506374661360_0006
17/09/26 00:35:08 INFO impl.YarnClientImpl: Submitted application application_1506374661360_0006
17/09/26 00:35:08 INFO mapreduce.Job: The url to track the job: http://newship:8088/proxy/application_1506374661360_0006/
17/09/26 00:35:08 INFO mapreduce.Job: Running job: job_1506374661360_0006
17/09/26 00:35:13 INFO mapreduce.Job: Job job_1506374661360_0006 running in uber mode : false
17/09/26 00:35:13 INFO mapreduce.Job:  map 0% reduce 0%
17/09/26 00:35:17 INFO mapreduce.Job:  map 30% reduce 0%
17/09/26 00:35:20 INFO mapreduce.Job:  map 50% reduce 0%
17/09/26 00:35:21 INFO mapreduce.Job:  map 60% reduce 0%
17/09/26 00:35:23 INFO mapreduce.Job:  map 80% reduce 0%
17/09/26 00:35:24 INFO mapreduce.Job:  map 90% reduce 0%
17/09/26 00:35:26 INFO mapreduce.Job:  map 100% reduce 0%
17/09/26 00:35:27 INFO mapreduce.Job:  map 100% reduce 100%
17/09/26 00:35:27 INFO mapreduce.Job: Job job_1506374661360_0006 completed successfully
17/09/26 00:35:27 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=226
		FILE: Number of bytes written=1563067
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2680
		HDFS: Number of bytes written=215
		HDFS: Number of read operations=43
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=3
	Job Counters
		Launched map tasks=10
		Launched reduce tasks=1
		Data-local map tasks=10
		Total time spent by all maps in occupied slots (ms)=22167
		Total time spent by all reduces in occupied slots (ms)=3017
		Total time spent by all map tasks (ms)=22167
		Total time spent by all reduce tasks (ms)=3017
		Total vcore-milliseconds taken by all map tasks=22167
		Total vcore-milliseconds taken by all reduce tasks=3017
		Total megabyte-milliseconds taken by all map tasks=42117300
		Total megabyte-milliseconds taken by all reduce tasks=5732300
	Map-Reduce Framework
		Map input records=10
		Map output records=20
		Map output bytes=180
		Map output materialized bytes=280
		Input split bytes=1500
		Combine input records=0
		Combine output records=0
		Reduce input groups=2
		Reduce shuffle bytes=280
		Reduce input records=20
		Reduce output records=0
		Spilled Records=40
		Shuffled Maps =10
		Failed Shuffles=0
		Merged Map outputs=10
		GC time elapsed (ms)=823
		CPU time spent (ms)=5510
		Physical memory (bytes) snapshot=2988027904
		Virtual memory (bytes) snapshot=21785776128
		Total committed heap usage (bytes)=2157445120
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters
		Bytes Read=1180
	File Output Format Counters
		Bytes Written=97
Job Finished in 20.242 seconds
Estimated value of Pi is 3.15000000000000000000
```
