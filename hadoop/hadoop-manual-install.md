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
    <property><name>yarn.app.mapreduce.am.resource.mb</name><value>512</value></property>
    <property><name>mapreduce.jobhistory.address</name><value>master.local:10020</value></property>
    <property><name>mapreduce.jobhistory.webapp.address</name><value>master.local:19888</value></property>
</configuration>
```

```etc/hadoop/yarn-site.xml:```:
```
<configuration>
  <property><name>yarn.resourcemanager.hostname</name><value>$YOUR_FQDN</value></property>
  <property><name>yarn.scheduler.minimum-allocation-mb</name><value>32</value></property>
  <property><name>yarn.scheduler.maximum-allocation-mb</name><value>1024</value></property>
  <property><name>yarn.scheduler.minimum-allocation-vcores</name><value>1</value></property>
  <property><name>yarn.scheduler.maximum-allocation-vcores</name><value>1</value></property>
  <property><name>yarn.nodemanager.resource.memory-mb</name><value>1024</value></property>
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
$ hadoop-daemon.sh start namenode
$ hadoop-daemon.sh start datanode
$ yarn-daemon.sh start resourcemanager
$ yarn-daemon.sh start nodemanager
$ mr-jobhistory-daemon.sh start historyserver
```