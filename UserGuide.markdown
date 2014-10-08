# Welcome to the glusterfs-hadoop user guide #

Most components within a typical Hadoop Distribution work out of the box with GlusterFS. This page lists any components that require additional configuration. If the component is not listed here, you can simply follow the instructions for using the component that are provided by your Hadoop Distribution provider.

## MapReduce Client tools: Pig,Flume,Sqoop,Hive

Easy install of Pig, Sqoop, Flume and Hive

Pig, Sqoop, Flume, and Hive can all be run locally:  They don't require use of any file system specific configuration.  The reason is that these tools invoke MapReduce services superficially to do their main hadoop related tasks.

That said, when you run tasks with these tools, you will want to make sure you use the glusterfs:/// file URI for your jobs.

Of course - this is not a hard requirement - the file system will still use glusterfs by default if you configure fs.defaultFS=glusterfs:///,

After that, you can follow specific instructions (i.e. setting up a distributed hive metastore or a coding up set of flume sinks/sources) for your particular needs.  The only thing to note is that, if you have any file system specific paths (for example in a flume hdfs sink, or in a hive.ql file), you should qualify them as "glusterfs:///" instead of using the typical "hdfs://x.y.z.a/..." scheme.  

To put these tools on your system so you can run pig/mahout/hive/sqoop jobs, you directly yum install them on any system which has added the CDH5 yum repositories.

    yum install pig
    yum install mahout
    yum install hive
    yum install sqoop

##Server side Ecosystem components

To install these tools, assuming you've added the CDH repos (you would have done so in the steps above), just run "yum install *".  

Each installation below requires slightly more sophisticated setup, because these tools run their own services.  Thus, you will need to do a couple of steps to add the glusterfs-hadoop connector to the right place.  

* [Oozie](https://forge.gluster.org/hadoop/pages/Additional_Components#Using+Oozie)
* [HBase](https://forge.gluster.org/hadoop/pages/Additional_Components#Using+HBase)
* [Solr](https://forge.gluster.org/hadoop/pages/Additional_Components#Using+Solr)
* [Impala](https://forge.gluster.org/hadoop/pages/Additional_Components#Using+Impala)
