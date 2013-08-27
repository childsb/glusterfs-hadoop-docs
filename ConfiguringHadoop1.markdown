**Download and Extract Hadoop**


Navigate to [Hadoop Download Page](http://hadoop.apache.org/releases.html#Download) and download the hadoop-1.2.1.tar.gz  tar-ball (the current stable 1.x release at the time this document was created) to the /opt directory on the JobTracker server. Extract the tar ball in the /opt directory.

For the sake of this document $HADOOP_HOME is the directory the tarball extracts to under /opt/

**Configure the Plugin**

On each node with the cluster, copy the plugin to the $HADOOP_HOME/lib directory.

** Modify the core-site.xml file **
Edit the $HADOOP_HOME/conf/core-site.xml file to include the following (additional properties can be added)

(note: node-1 needs to be replaced by a hostname from your storage pool):

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

`<configuration>`
` <property>`
`  <name>fs.defaultFS</name>`
`  <value>glusterfs://node-1:9000</value>`
` </property>`

` <property>`
`  <name>fs.default.name</name>`
`  <value>glusterfs://node-1:9000</value>`
` </property>`

` <property>`
`  <name>fs.AbstractFileSystem.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.local.GlusterFs</value>`
` </property>`

` <property>`
`  <name>fs.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.glusterfs.GlusterFileSystem</value>`
` </property>`

` <property>`
`  <name>fs.glusterfs.mount</name>`
`  <value>/mnt/glusterfs</value>`
` </property>`

` <property>`
`  <name>fs.glusterfs.server</name>`
`  <value>node-1</value>`
` </property>`

`</configuration>`

**Modify the Slaves files**

Navigate to the $HADOOP_HOME/conf directory. Modify the masters file to list the server you have designated as the JobTracker. Modify the slaves files to list all the servers within the storage pool for your volume, with each one on a new line.

** Synchronize the configuration across the cluster **

SCP the $HADOOP_HOME directory to each server in the  the cluster, for example:
    scp -r $HADOOP_HOME/ root@svr2:/opt/
    scp -r $HADOOP_HOME/ root@svr3:/opt/
    scp -r $HADOOP_HOME/ root@svr4:/opt/
    etc...

**Starting Hadoop**

On the JobTracker server, open a terminal window, navigate to the $HADOOP_HOME directory and start Hadoop by running:

     bin/start-mapred.sh 

This script  inspects the masters and slaves files in the $HADOOP_HOME/conf directory and starts the JobTracker on the Master server and starts the TaskTracker process on all the slave servers.

**Verifying Hadoop is running successfully on GlusterFS**

In a browser, launch the Hadoop JobTracker UI by navigating to http://<JobTrackerHostName>:50030 
Under the TaskTracker nodes, verify that all the hosts listed within the Hadoop Slaves have reported in. If a host is missing, 
Go to JobTracker, verify that all the TaskTrackers reported in.