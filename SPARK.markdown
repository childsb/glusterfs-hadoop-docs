#Deploy SPARK on GlusterFS#

This guide will show you how to run SPARK on GlusterFS using the glusterfs-hadoop plugin.


##Solution Components

* GlusterFS 3.3+
* Apache Hadoop 1.x or 2.x
* glusterfs-hadoop Hadoop plugin
* JRE 1.6+
* SPARK 1.1.0

##Pre-Req

* Install Hadoop and configure for Gluster using this guide: [Configuring Hadoop For GlusterFS](https://forge.gluster.org/hadoop/pages/Configuration) 

##SPARK Config
For each Node in the cluster:

* Download and unzip SPARK to `/opt/spark-<VERSION>`

* Edit `<SPARK_INSTALL>/conf/master` to include the IP address of the SPARK master node

* Edit `<SPARK_INSTALL>/conf/slaves` to include the IP addresses of each slave node in the SPARK cluster

* Create or modify `<SPARK_INSTALL>/conf/spark-env.sh` to include an environment variable pointing to the Hadoop installation setup previously and a pointer to the Hadoop ./lib/ directory for the glusterfs-hadoop plugin JAR : 

`HADOOP_CONF_DIR=<HADOOP_INSTALL>/etc/hadoop`

`SPARK_CLASSPATH=<HADOOP_INSTALL>/share/common/lib/*:$SPARK_CLASSPATH`

##Using SPARK on GlusterFS

Use the glusterfs URI to access gluster in SPARK:

`JavaRDD<String> distFile = sc.textFile("glusterfs:///anyTextFile.txt");`
