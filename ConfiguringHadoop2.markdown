**Download and Extract Hadoop**

Navigate to [Hadoop Download Page](http://hadoop.apache.org/releases.html#Download) and download the hadoop-2.1.0-beta.tar.gz  tar-ball (the most recent 2.x release at the time this document was created) to the /opt directory on the Master server. Extract the tar ball in the /opt directory.

For the sake of this document $HADOOP_HOME is the directory the tarball extracts to under /opt/

**Configure the Plugin**

Copy the plugin jar to $HADOOP_HOME/share/hadoop/common/lib/. [The plugin jar can be obtained here](http://rhbd.s3.amazonaws.com/maven/indexV2.html)

** Modify the hadoop-env.sh file **

Edit the $HADOOP_HOME/etc/hadoop/hadoop-env.sh file to reflect JAVA_HOME. Remove the # preceding the line and set the JAVA_HOME to the correct directory. This would usually be:

 export JAVA_HOME=/usr/lib/jvm/java-1.6.0/

** Modify the mapred-site.xml **

Navigate to $HADOOP_HOME/etc/hadoop and modify the mapred-site.xml to reflect the following:

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`
`<property>`
`<name>yarn.nodemanager.linux-container-executor.group</name>`
`<value>hadoop</value>`
`</property>`

`<property>`
`<name>yarn.nodemanager.container-executor.class</name> `
`<value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>`
`</property>`
` <configuration>`
`   <property>`
`     <name>mapreduce.framework.name</name>`
`     <value>yarn</value>`
`   </property>`

`  <property>`
`    <name>yarn.app.mapreduce.am.staging-dir</name>`
`    <value>glusterfs:///user</value>`
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
`  <value>glusterfs:///</value>`
` </property>`

` <property>`
`  <name>fs.default.name</name>`
`  <value>glusterfs:///</value>`
` </property>`

` <property>`
`  <name>fs.AbstractFileSystem.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.local.GlusterFs</value>`
` </property>`

` <property>`
`  <name>fs.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.glusterfs.GlusterFileSystem</value>`
` </property>`

`<!-- two volume setup. gv0 and gv1 -->`
`<property>`
`<name>fs.glusterfs.volumes</name>`
`<value>gv0,gv1</value>`
`</property>`

`<property>`
`<name>fs.glusterfs.volume.fuse.gv0</name>`
`<value>/mnt/gv0</value>`
`</property>`

`<property>`
`<name>fs.glusterfs.volume.fuse.gv1</name>`
`<value>/mnt/gv1</value>`
`</property>`


`</configuration>`


** Modify the yarn-site.xml **

Navigate to $HADOOP_HOME/etc/hadoop) and modify the yarn-site.xml to reflect the following (note: node-1 needs to be replaced with the server designated as your ResourceManager (Master) in your storage pool):

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

`<configuration>`
`<property>`
`    <name>yarn.nodemanager.aux-services</name>`
`    <value>mapreduce_shuffle</value>`
`    <description>Auxilliary services of NodeManager</description>`
`  </property>`

`  <property>`
`    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>`
`    <value>org.apache.hadoop.mapred.ShuffleHandler</value>`
 ` </property>`

`  <property>`
`    <name>yarn.resourcemanager.resource-tracker.address</name>`
`    <value>node-1:8025</value>`
`  </property>`

`  <property>`
`    <name>yarn.resourcemanager.scheduler.address</name>`
`    <value>node-1:8030</value>`
`  </property>`

`  <property>`
`    <name>yarn.resourcemanager.address</name>`
`    <value>node-1:9040</value>`
`  </property>`
` </configuration>`

**Configure for Multiple Users**

Setup container executor by creating the following script (in $HADOOP_HOME):

---------------------------

    #!/bin/sh
    HADOOP_HOME=/opt/hadoop
    process_user=yarn
    process_group=hadoop
    task_controller=$HADOOP_HOME/bin/container-executor
    task_cfg=$HADOOP_HOME/etc/hadoop/container-executor.cfg

    echo "Configuring the Linux Container Executor for Hadoop"
    chown root:${process_group} ${task_controller} 
    chmod 6050 ${task_controller}
    chown root:${process_group} ${task_cfg}
    chown -R ${process_user}:${process_group} ${HADOOP}/logs

------------------------------

Set the $HADOOP_HOME/etc/hadop/container-executor.cfg to:
    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom

** Synchronize the configuration across the cluster **

SCP the $HADOOP_HOME directory to each server in the  the cluster, for example:
    scp -r $HADOOP_HOME/ root@svr2:/opt/
    scp -r $HADOOP_HOME/ root@svr3:/opt/
    scp -r $HADOOP_HOME/ root@svr4:/opt/
    etc...

Then ssh to each node, and run the container-executor script.



**Starting Hadoop**

On the Master server, open a terminal window, navigate to $HADOOP_HOME and run the following commands:
    sbin/yarn-daemon.sh start resourcemanager
    sbin/yarn-daemon.sh start nodemanager
    sbin/mr-jobhistory-daemon.sh start historyserver

On each Slave server, open a terminal window, navigate to $HADOOP_HOME and run the following commands:

    sbin/yarn-daemon.sh start nodemanager

**Verifying Hadoop is running successfully on GlusterFS**

In a browser, launch the YARN ResourceManager UI by navigating to http://<$MasterHostName>:8088/cluster/nodes

Verify that all the hosts listed within your storage pool have reported in. If a host is missing, shell into that host and take a look at the files within $HADOOP_HOME/logs to check for errors when the process attempted to start.

** Running a sample Hadoop Job **

Navigate to $HADOOP_HOME and run the following:

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar teragen 10000 in-dir

and then once TeraGen has complete, run

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar terasort in-dir out-dir