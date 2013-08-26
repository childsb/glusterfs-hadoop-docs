## Pre-Requisites ##

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* A GlusterFS 3.3 +
* Apache Hadoop 1.x or 2.x
* Oracle Java Runtime Environment (JRE) 1.6 +
* FUSE Kernel Patches applied to all GlusterFS nodes

## Deployment Architecture ##

The deployment process involves initially installing and configuring GlusterFS on a cluster of servers, building your trusted storage pool and creating your gluster volume. Once that has been done you need to set the following appropriate parameters on the gluster volume to ensure consistency across the namespace:
gluster volume set HadoopVol quick-read off
gluster volume set HadoopVol cluster.eager-lock on
gluster volume set HadoopVol performance.stat-prefetch off

Mount appropriately

Installing the FUSE patch and Java

Install Hadoop. Tarball Option 

**For Hadoop 1.x:**
* Install a TaskTracker on every node within your Trusted Storage Pool. 
* Copy the plugin to ...
* Modify the core-site.xml to reflect:

**For Hadoop 2.x:**
* install a NodeManager on every node within your Trusted Storage Pool. 
* Copy the plugin to ...
* Modify the core-site.xml to reflect:




## Configuration ##

  All plugin configuration is done in a single XML file (core-site.xml) with <name><value> tags in each <property>
  block.

  Brief explanation of the tunables and the values they accept (change them where-ever needed) are mentioned below

  name:  fs.glusterfs.impl
  value: org.apache.hadoop.fs.glusterfs.GlusterFileSystem

         The default FileSystem API to use (there is little reason to modify this).

  name:  fs.default.name
  value: glusterfs://server:port

         The default name that hadoop uses to represent file as a URI (typically a server:port tuple). Use any host
         in the cluster as the server and any port number. This option has to be in server:port format for hadoop
         to create file URI; but is not used by plugin.

  name:  fs.glusterfs.volname
  value: volume-dist-rep

         The volume to mount.


  name:  fs.glusterfs.mount
  value: /mnt/glusterfs

         This is the directory that the plugin will use to mount (FUSE mount) the volume.

  name:  fs.glusterfs.server
  value: 192.168.1.36, hackme.zugzug.org

         To mount a volume the plugin needs to know the hostname or the IP of a GlusterFS server in the cluster.
         Mention it here.


## Usage ##

  Once configured, start Hadoop Map/Reduce daemons

  # cd $HADOOP_HOME
  # ./bin/start-mapred.sh

  If the map/reduce job/task trackers are up, all I/O will be done to GlusterFS.