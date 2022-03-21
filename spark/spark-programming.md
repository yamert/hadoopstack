# Programming
## Dependencies

Scala dependency
```
libraryDependencies += "org.apache.spark" % "spark-core_2.11" % "2.2.0" % "provided"
```

https://github.com/saurfang/sbt-spark-submit
```
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-yarn" % "1.4.0" % "provided" excludeAll ExclusionRule(organization = "org.apache.hadoop"),
  "org.apache.hadoop" % "hadoop-client" % "2.4.0" % "provided",
  "org.apache.hadoop" % "hadoop-yarn-client" % "2.4.0" % "provided"
)
```


## Sources
```
git clone git://github.com/apache/spark.git -b branch-2.2
```


## Links
https://github.com/saurfang/sbt-spark-submit
https://github.com/databricks/sbt-spark-package

