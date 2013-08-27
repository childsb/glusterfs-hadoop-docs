**Download and Extract Hadoop**

Navigate to [Hadoop Download Page](http://hadoop.apache.org/releases.html#Download) and download the hadoop-2.1.0-beta.tar.gz  tar-ball (the most recent 2.x release at the time this document was created) to the /opt directory on the Master server. Extract the tar ball in the /opt directory.

For the sake of this document $HADOOP_HOME is the directory the tarball extracts to under /opt/

**Configure the Plugin**

On each node within your storage pool, copy the plugin to $HADOOP_HOME/share/hadoop/common/lib/

** Modify the mapred-site.xml **

Navigate to $HADOOP_HOME/etc/hadoop and modify the mapred-site.xml to reflect the following:

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

` <configuration>`
`   <property>`
`     <name>mapreduce.framework.name</name>`
`     <value>yarn</value>`
`   </property>`

`  <property>`
`    <name>yarn.app.mapreduce.am.staging-dir</name>`
`    <value>glusterfs:///tmp/hadoop-yarn/staging/mapred/.staging</value>`
`  </property>`

`  <property>`
`    <name>mapred.healthChecker.script.path</name>`
`    <value>glusterfs:///mapred/jobstatus</value>`
`  </property>`

`  <property>`
`    <name>mapred.job.tracker.history.completed.location</name>`
`    <value>glusterfs:///mapred/history/done</value>`
`  </property>`

`  <property>`
`    <name>mapred.system.dir</name>`
`    <value>glusterfs:///mapred/system</value>`
`  </property>`

`  <property>`
`    <name>mapreduce.jobtracker.staging.root.dir</name>`
`    <value>glusterfs:///user</value>`
`  </property>`

`</configuration>`

** Modify the core-site.xml **

Navigate to $HADOOP_HOME/etc/hadoop) and modify the core-site.xml to reflect the following (note: node-1 needs to be replaced a server in your storage pool):

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

**Starting Hadoop**

On the JobTracker server, open a terminal window, navigate to the $HADOOP_HOME directory and start Hadoop by running:
     bin/start-mapred.sh 

**Verifying Hadoop is running successfully on GlusterFS**

In a browser, launch the Hadoop JobTracker UI by navigating to http://<JobTrackerHostName>:50030 
Under the TaskTracker nodes, verify that all the hosts listed within the Hadoop Slaves have reported in. If a host is missing, 
Go to JobTracker, verify that all the TaskTrackers reported in.