## Pre-Requisites ##

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* A GlusterFS 3.3 +
* Apache Hadoop 1.x or 2.x
* Oracle Java Runtime Environment (JRE) 1.6 +
* FUSE Kernel Patches applied to all GlusterFS nodes

## Configuration Guide ##

1) The deployment process involves initially installing and configuring GlusterFS on a cluster of servers, building your trusted storage pool and creating your gluster volume. Once that has been done you need to set the following appropriate parameters on the gluster volume to ensure consistency across the namespace. This example assumes your Gluster volume is called "HadoopVol":

gluster volume set HadoopVol quick-read off
gluster volume set HadoopVol cluster.eager-lock on
gluster volume set HadoopVol performance.stat-prefetch off

2) Once the GlusterFS volume is appropriately configured one needs to install the FUSE Kernel patch on every node within the storage pool. This patch resolves a current issue with Linux whereby the inode cache becomes stale and causes namespace consistency issue in parallel environments. We are working on providing a link to the RPM download. Since this is a Kernel patch, it is required that you reboot after installing it.

3) Install Oracle Java 1.6

4) Mount the Gluster volume to /mnt/glusterfs on every node within the trusted storage pool. Please note that this is a specialized mount command that sets the attribute and entry timeouts to zero. This is also required for namespace consistency and related to the FUSE patch. It is recommended that you take measures to ensure the mount is persisted upon reboot.

glusterfs --attribute-timeout=0 --entry-timeout=0 --volfile-id=/HadoopVol --volfile-server=<HOST_NAME> /mnt/glusterfs

5) Install Hadoop

**For Hadoop 1.x:**
* Install a JobTracker on a single designated node within your storage pool
* Install a TaskTracker on every node within your Trusted Storage Pool. 
* On each node within your storage pool, copy the plugin to $HADOOP_HOME/lib/

**For Hadoop 2.x:**
* Install a ResourceManager and JobHistoryServer on a single designated node within your Trusted Storage Pool
* install a NodeManager on every node within your storage pool. 
* On each node within your storage pool, copy the plugin to $HADOOP_HOME/share/hadoop/common/lib/
* Modify the mapred-site.xml to reflect the following:

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

`  <property>
`    <name>mapreduce.jobtracker.staging.root.dir</name>`
`    <value>glusterfs:///user</value>`
`  </property>`

`</configuration>`

**For both** Hadoop 1.x ($HADOOP_HOME/conf) or Hadoop 2.x ($HADOOP_HOME/etc/hadoop), modify the core-site.xml to reflect the following:

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

`<configuration>`
` <property>`
`  <name>fs.defaultFS</name>`
`  <value>glusterfs://server:9000</value>`
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

## Usage ##

  Once configured, start Hadoop Map/Reduce daemons

  # cd $HADOOP_HOME
  # ./bin/start-mapred.sh

  If the map/reduce job/task trackers are up, all I/O will be done to GlusterFS.