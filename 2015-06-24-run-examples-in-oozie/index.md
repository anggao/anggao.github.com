# Run Examples in Oozie


- Extract Examples Tarball file

```shell
cd $OOZIE_HOME
tar -xzvf oozie-examples.tar.gz
```

- Edit job.properties file

```shell
cd examples/apps/map-reduce
vim job.properties

    nameNode=hdfs://YARN001:8020
    jobTracker=YARN001:8032
    queueName=default
    examplesRoot=examples
```

- Copy examples into HDFS

```shell
hadoop fs -put examples examples
cd $OOZIE_HOME
oozie job -oozie http://localhost:11000/oozie -config $OOZIE_HOME/examples/apps/map-reduce/job.properties -run
```

