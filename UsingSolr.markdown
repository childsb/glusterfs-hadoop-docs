**Directions for Running CDH5 Solr**


The simplest way to start with SOLR is using a tarball.  For this example, we will download the CDH5 Tarball and untar it to /opt/solr.

The SOLR tarball can be acquired from the CDH5 tarball downloads page:

http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH-Version-and-Packaging-Information/cdhvd_cdh_package_tarball.html

Extract the tarball, and go to the examples/ directory, where you can launch an embedded core.

Now, cd into /opt/solr, to the examples/directory.

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

*Testing SOLR with the embedded examples*

Now that solr is running, you can test it by running the examples which are embedded in the distribution. 

` cd /opt/solr-4.4.0-cdh5.0.2/example/exampledocs && curl http://mrg42:8983/solr/update?commit=true -H "Content-Type: text/xml" -d "@mem.xml" `

