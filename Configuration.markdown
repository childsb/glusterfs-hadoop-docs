#Deploying a Working Cluster#

##Solution Components

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* GlusterFS 3.3 +
* FUSE Kernel Patch
* Apache Hadoop 1.x or 2.x
* The GlusterFS Hadoop FileSystem Plugin
* Oracle Java Runtime Environment (JRE) 1.6 +
* NTP

##Configuration Guide

This guide is focused on giving the community a way to quickly evaluate Hadoop on GlusterFS. A popular way to try the solution out is to set up 4 Virtual Machines with Fedora 19 and then follow the instructions below.

** Installing and Configuring GlusterFS ** 

Prior to Installing and Configuring Hadoop, one needs to first install GlusterFS across the cluster and configure a Distributed Replicated 2 GlusterFS volume.  This is easy to do [with our automated deployment tool](https://forge.gluster.org/hadoop/pages/GlusterfsClusterInstall). You can also [do this manually](https://forge.gluster.org/hadoop/pages/InstallingAndConfiguringGlusterFS) 

Additional Gluster Documentation and Downloads [are available here](http://www.gluster.org/download/)

** Install Oracle Java 1.6 **

This is a requirement of Hadoop. Hadoop has not yet been widely tested with OpenJDK, although it may well work. The JRE needs to be installed on every server within the trusted storage pool that your Gluster Volume uses. [Download it here](http://www.oracle.com/technetwork/java/javase/downloads/jdk6u38-downloads-1877406.html)

** Configuring Hadoop related tools **

**For Hadoop 2.x:** 
See - [Configuring Hadoop 2.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop2) for GlusterFS

**For Cloudera CDH5:** 
See - [Configuring CDH 5](https://forge.gluster.org/hadoop/pages/Cloudera) for GlusterFS

**For SPARK 1.1.0:** 
See - [Configuring SPARK](https://forge.gluster.org/hadoop/pages/SPARK110) for GlusterFS

**User Guide:** If you have questions about using specific components within the Hadoop Ecosystem, [please see the user guide](https://forge.gluster.org/hadoop/pages/UserGuide)

**Sync Clocks**
		
It is important that the time and date be synchronized for each server within the cluster.   The Network Time Protocol (NTP) is a popular method to achieve this. 
		
To install NTP on each node run:
		
   yum install ntp
		
To update the clock on each machine run:
		     
    ntpd -qg
		
Consult the ntpd documentation to configure periodic time resync.

** Troubleshooting Tips ** 
[Troubleshooting Hadoop Install](https://forge.gluster.org/hadoop/pages/TroubleShootHadoop)

-------------------------------------

# Securing your Hadoop Cluster #
Hadoop can run in multiple configurations with varying security.  Each section below outlines additional setup configuration to secure your cluster.

##Run-as-root
This is the least secure method.  All services are started as root, and map reduce jobs are launched by root user.  This is a security risk don't do it.

##Single (Mapred or Yarn) User
The single tenant non-root setup is covered below in the [Basic Hadoop Setup](https://forge.gluster.org/hadoop/pages/Configuration#Basic+Hadoop+Setup) section.

##Multi-User

What if we want our users who SUBMIT jobs to be different than the yarn daemon?  In this case, we use ACLs, which will by default add the running daemon permissions to read files which have been written by any user on the system which writes to a particular directory. 

Thus, in order for hadoop to run in full multi-user mode, a special user has to be designated to run the hadoop daemons (typically, this special user already exists on a production cluster, and is either called "yarn" or "mapred").  This user may be restricted from logging in, or accessing the entire file system.  However, the user must be granted read and write permission to the map/reduce staging directory.  This is accomplished using POSIX ACLs.  To enable ACLs on your gluster volume, mount it with the ACL flag (on every node in the cluster).

**fstab entry for Gluster Volume w/ACL**
`localhost:/gv0 /mnt/glusterfs glusterfs acl,auto,transport=tcp 0 0` 

**Create the Hadoop Group**
`groupadd -g 500 hadoop`

**Create the Hadoop Daemon User**
`adduser --no-create-home --system --uid 1000 --gid 500 yarn`

**Create Hadoop Users**
Hadoop users can be created like other users (LDAP, command line, etc) and require no special configuration

**Specify the Hadoop Daemon User**
The GlusterFS plugin will add the hadoop daemon user to certain system directories to allow access across the cluster.  It's important that the staging directory and the Hadoop Daemon user are supplied in configuration:

**In core-site.xml**:
`  <property>`
`    <name>gluster.daemon.user</name>`
`    <value>yarn</value>`
`  </property>`

_Additional For Hadoop 2.x:_

**In yarn-site.xml**:

`  <property>`
`   <name>yarn.app.mapreduce.am.staging-dir</name>    `
`   <value>glusterfs:///job-staging-yarn</value> `
`   </property>`

`   <property>`
`     <name>yarn.nodemanager.linux-container-executor.group</name>`
`      <value>hadoop</value>`
`    </property>`

`<property>`
`  <name>yarn.nodemanager.container-executor.class</name>`	
 `<value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>`
`</property>`


**In mapred-site.xml**
`  <property>`
`    <name>mapreduce.jobhistory.done-dir</name>`
`    <value>glusterfs:///mr_history/done</value>`
`  </property>`
  
`  <property>`
`    <name>mapreduce.jobhistory.intermediate-done-dir</name>`
`    <value>glusterfs:///mr_history/tmp</value>`
`  </property>`

**Linux Task Controller**

Setup the LinuxTaskController on each node.  Details can be found here:
[Securing Hadoop](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html#Running_Hadoop_in_Secure_Mode)

**In container-executor.cfg**
`   yarn.nodemanager.linux-container-executor.group=hadoop`
`   banned.users=yarn`
`   min.user.id=1000`
`   allowed.system.users=mapred`

**Set the UID bit on Container Executor**
`  chown root:hadoop $HADOOP_INSTALL/bin/container-executor`
`  chmod 6050 $HADOOP_INSTALL/bin/container-executor`

**Running map/reduce Jobs**
Ensure that your Hadoop Cluster users have appropriate permissions to the Gluster Volume.  This can be accomplished with permissions/mode on data files and output directories or ACL.  **Users must have read to data files on the volume and write access to output directories.**  

Once the user has permissions to the required input and output directories, login to the cluster as that user and submit the job.


----------------------------------------
# Basic Hadoop Setup #
The following steps provide a guide on how to configure Hadoop to run under the "mapred" user account. 

**Assumptions:**

*  the Hadoop tarball has been extracted to /opt/hadoop-1.2.1 or relevant version
*  on each server, the brick has been mounted as /mnt/brick1
*  mapred.local.dir is set to /mnt/brick1/mapredlocal within conf/mapred-site.xml

**1. Create the mapred user and add it to the hadoop group**

Run the following commands as root:

`groupadd hadoop`
`useradd -g hadoop mapred`

**2. Set the permissions for Hadoop**

Run the following commands as root:

`chmod 777 /opt/hadoop-1.2.1/*.jar`

`mkdir -p /mnt/brick1/mapredlocal`
`chown -R mapred:hadoop /mnt/brick1/mapredlocal`
`chmod -R 755 /mnt/brick1/mapredlocal`

`chmod a+x /opt/hadoop-1.2.1/conf/`
`chown -R mapred:hadoop /opt/hadoop-1.2.1/conf/../`
`chmod -R 755 /opt/hadoop-1.2.1/conf/../`

**3. Set the permissions for getfattr**

As root, create a sudoers file for gluster using the following command:
`vi /etc/sudoers.d/gluster`

Press "i" to switch to insert mode and paste in the following line:
`mapred ALL= NOPASSWD: /usr/bin/getfattr`

Hit escape and type ":wq" and hit enter to save and exit. 

As root, run the following command:
`chmod 440 /etc/sudoers.d/gluster`

**4. Set the permissions for the gluster mount**

As root, run the following command:
`chmod -R 1777 /mnt/glusterfs`
A quick note for those automating this: For (relatively) obvious reasons, the above "chmod" command must be run AFTER you have run the "mount ..." command to mount glusterfs' FUSE mount on your local machine.  

Switch to the mapred user and create the Hadoop System Directory by running the following commands:
`su mapred`
`mkdir -p /mnt/glusterfs/mapred/system`

**5. Start the JobTracker and TaskTracker under the mapred user**

`su mapred`
`bin/hadoop-daemon.sh --config /opt/hadoop-1.2.1/conf/ start jobtracker`
`bin/hadoop-daemon.sh --config /opt/hadoop-1.2.1/conf/ start tasktracker`

**6. Submitting Jobs**

Once you have followed the steps above, you need to switch to the mapred user prior to submitting any jobs. For example:
 `su mapred` 
` bin/hadoop jar hadoop-examples-1.2.1.jar teragen 1000 in-dir`
