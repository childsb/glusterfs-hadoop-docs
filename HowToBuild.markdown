## Pre-Requisites ##

- You *should* have a working gluster mount for unit tests, although they may feasibly pass without a gluster mount since the proxy writes into a local file system, at least for versions 1.x and 2.x of the plugin.

- [Maven] (supported version 3.1, the build is known to fail on older maven/surefire combinations which seem not to support @Ignore annotations - currently investigating this).  (http://maven.apache.org/).

- JDK  1.6+.  OpenJDK is fine for development.   Different deployments may use different vendor specific JVM's, however, so you should always test your MapReduce code according to whats on your cluster.

## ECLIPSE DEVELOPMENT ##

1) Setup gluster mount /mnt/glusterfs or whatever. 

    mount -t glusterfs -o acl localhost:/HadoopVol /mnt/glusterfs 
   
2) To run any unit tests, or all unit tests click your project, click on "Run As", click "Run Configurations", and add these parameters as "VM Arguments"

-DHCFS_FILE_SYSTEM_CONNECTOR=org.apache.hadoop.fs.test.connector.glusterfs.GlusterFileSystemTestConnector 

-DHCFS_CLASSNAME=org.apache.hadoop.fs.glusterfs.GlusterFileSystem 

-DHCFS_CLASSNAME=org.apache.hadoop.fs.glusterfs.GlusterFileSystem 

-DGLUSTER_MOUNT=/mnt/glusterfs

## Building with MAVEN ##

export HCFS_FILE_SYSTEM_CONNECTOR="org.apache.hadoop.fs.test.connector.glusterfs.GlusterFileSystemTestConnector" 
export DHCFS_CLASSNAME="org.apache.hadoop.fs.glusterfs.GlusterFileSystem" 
export HCFS_CLASSNAME="org.apache.hadoop.fs.glusterfs.GlusterFileSystem" 
export GLUSTER_MOUNT="/mnt/glusterfs"

And then run: 

mvn package; 

You will see the shim written out to the target/ folder.  