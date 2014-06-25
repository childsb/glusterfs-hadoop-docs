**Directions for Running CDH5 Solr**


The simplest way to start with SOLR is using a tarball, for example, the ones distributed by the ASF.

Extract the tarball, and go to the examples/ directory, where you can launch an embedded core.

Then, find the solrconfig.xml file.  Edit it to contain the following xml: 

    <directoryFactory name="DirectoryFactory" class="org.apache.solr.core.HdfsDirectoryFactory">
    <str name="solr.hdfs.home">glusterfs:///solr</str>
    <str name="solr.hdfs.confdir">/etc/hadoop/conf</str>
    </directoryFactory>

This tells solr, when launched, to use gluster as the underlying file store.   You also should make sure that **glusterfs-hadoop jar**, and  the **hadoop-common jar**, are both on the classpath of the solr web server.  To do so - you can update the solrconfig.xml jar directives.  You can also copy the jars in at runtime, to be really safe. 

Once your example core with gluster configuration is setup, launch it with the following properties: 

    java -Dsolr.directoryFactory=HdfsDirectoryFactory -Dsolr.lock.type=hdfs -Dsolr.data.dir=glusterfs:///solr -Dsolr.updatelog=glusterfs:///solr -Dlog4j.configuration=file:/opt/solr-4.4.0-cdh5.0.2/example/etc/logging.properties -jar start.jar 


This starts a basic SOLR server on port 8983.

From here, you will want to make sure you are plugged into the gluster file system for backend storage. 

If the launch works, then you will be able to see a healthy SOLR core running on `http://<SERVER>:8983/solr/`
