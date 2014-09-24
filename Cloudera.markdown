# Setting up CDH5 #

All the apache components of CDH5, to our knowledge, are 100% HCFS compatible on glusterfs-hadoop from the basic testing that we've done.  Here is how to get started with a Cloudera's recent CDH5 release on hadoop.

####################
**For CDH5: Simple Security Mode ** 
See - [CDH5 (simple) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SIMPLE) for GlusterFS

**For CDH 5: Full security mode ** 
See - [CDH5 (kerberos) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SECURE) for GlusterFS

# Setting up the Hadoop Ecosystem on top of CDH5 #
######################

Common hadoop client applications (*Pig, Hive, Flume, Sqoop, Zookeeper, and Mahout*) all run as is, as hadoop clients, with no extra configuration required.  *Meanwhile*, the services which run on hadoop (i.e. *Oozie, HBase, Solr*  ) require more sohpisticated configuration steps, because you need to update certain files, and possibly add the glusterfs-hadoop plugin into directories other than the /usr/lib/hadoop/lib location.  

So lets get started.  

## Part 1: Hadoop Client apps on glusterfs ##

After you've yum installed the client apps (`yum install hive pig mahout flume-ng zookeeper sqoop`) you can use [Our backported BigTop Smoke Tests] (https://github.com/roofmonkey/simple-smokes/) to test them quite easily.  You will have to export some variables first.

    export HIVE_CONF_DIR=/etc/hive/conf/
    export PIG_HOME=/usr/lib/pig/
    export SQOOP_HOME=/usr/lib/sqoop/

And then you can run the test.sh file in the top directory.

## Part 2: Hadoop Service applications on glusterfs ##

We have specific directions for installing and running CDH5 ecosystem service applications below.

* [HBase](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+HBase). This page also includes instructions how to test Hbase.

* [Oozie](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+Oozie).  For testing Oozie, you can use https://forge.gluster.org/hadoop/oozie-smoke-test.

* [Solr](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+Solr).  This page has specific instructions on how to test Solr. 

If you have any issues, you can contact us directly or ping the gluster users mailing list.   


