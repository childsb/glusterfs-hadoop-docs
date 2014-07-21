# Setting up CDH5 #

All the apache components of CDH5, to our knowledge, are 100% HCFS compatible on glusterfs-hadoop from the basic testing that we've done.  Here is how to get started with a Cloudera's recent CDH5 release on hadoop.

**For CDH5: Simple Security Mode ** 
See - [CDH5 (simple) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SIMPLE) for GlusterFS

**For CDH 5: Full security mode ** 
See - [CDH5 (kerberos) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SECURE) for GlusterFS

# Setting up the Hadoop Ecosystem on top of CDH5 #

## Hadoop Client apps on glusterfs ##

*Pig, Hive, Flume and Mahout* all run as is, as hadoop clients, with no extra configuration required.  
 
To test these components and to better understand how to write applications that leverage hadoop on glusterfs, you can see our smoke test repository : [Backported BigTop Smoke Tests] (https://github.com/roofmonkey/simple-smokes/).  This repository takes the basic [Apache BigTop] (http://bigtop.apache.org/) smoke tests and back ports them into a simple and easy to use shell script which directly uses the gluster fuse mount in certain parts.

Some things to keep in mind:  

Its best to always use the "glusterfs:///" qualifier when creating hive and pig scripts which reference the file system (i.e. `create extrenal table `

## Hadoop Services ##

Services which run on hadoop (i.e. *Oozie, HBase, Solr*  ) require more sohpisticated configuration steps, because you need to update certain files, and possibly add the glusterfs-hadoop plugin into directories other than the /usr/lib/hadoop/lib location.  Thus, we have specific directions for installing and running CDH5 ecosystem service applications below.

* [HBase] (https://forge.gluster.org/hadoop/pages/UsingHBase). This page also includes instructions how to test Hbase.

* [Oozie] (https://forge.gluster.org/hadoop/pages/UsingOozie).  For testing Oozie, you can use https://forge.gluster.org/hadoop/oozie-smoke-test.

* [Solr] (https://forge.gluster.org/hadoop/pages/UsingSolr).  This page has specific instructions on how to test Solr. 

If you have any issues, you can contact us directly or ping the gluster users mailing list.   