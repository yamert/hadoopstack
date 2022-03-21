# Spark manual install
## Links
https://spark.apache.org/
https://spark.apache.org/downloads.html

### Download
- "Choose a Spark release": "2.2.0 (Jul 11 2017)"
- "Choose a package type" : "Pre-built for Apache Hadoop 2.7 and later"
Download spark-2.2.0-bin-hadoop2.7.tgz

### Release notes to Spark 2.2.0
https://spark.apache.org/releases/spark-release-2-2-0.html

### Documentation
https://spark.apache.org/docs/latest/api/scala/index.html#package


## Install process
```
$ cd ~/
$ pwd
/home/hadoop
$ tar -xf /path/to/spark-2.2.0-bin-hadoop2.7.tgz
$ ls -ld spark-2.2.0-bin-hadoop2.7
drwxr-xr-x 12 hadoop hadoop 4096 Jul  1 02:09 spark-2.2.0-bin-hadoop2.7
$ ln -sf spark-2.2.0-bin-hadoop2.7 spark
```