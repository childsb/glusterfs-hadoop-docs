# Securing your Hadoop Cluster
Hadoop can run in multiple configurations with varying security.  Each section below outlines additional setup configuration to secure your cluster.

## Run-as-root
This is the least secure method.  All services are started as root, and map reduce jobs are launched by root user.  This is a security risk don't do it.

## Single (Mapred or Yarn) User
The single tenant non-root setup is covered by the basic [configuration] (https://forge.gluster.org/hadoop/pages/Configuration).

## Multi-User

What if we want our users who SUBMIT jobs to be different than the yarn daemon ?  In this case, we use ACLs, which will by default add the running daemon permissions to read files which have been written by any user on the system which writes to a particular directory. 

Thus, in order for hadoop to run in full multi-user mode, a special user has to be designated to run the hadoop daemons (typically, this special user already exists on a production cluster, and is either called "yarn" or "mapred").  This user may be restricted from logging in, or accessing the entire file system.  However, the user must be granted read and write permission to the map/reduce staging directory.  This is accomplished using POSIX ACLs.  To enable ACLs on your gluster volume, mount it with the ACL flag (on every node in the cluster).

### fstab entry for Gluster Volume w/ACL
`localhost:/gv0 /mnt/glusterfs glusterfs acl,auto,transport=tcp 0 0` 

### Create the Hadoop Group
`groupadd -g 500 hadoop`

### Create the Hadoop Daemon User 
`adduser --no-create-home --system --uid 1000 --gid 500 yarn`

### Create Hadoop Users
Hadoop users can be created like other users (LDAP, command line, etc) and require no special configuration

### Specify the Hadoop Daemon User
The GlusterFS plugin will add the hadoop daemon user to certain system directories to allow access across the cluster.  It's important that the staging directory and the Hadoop Daemon user are supplied in configuration:

**In core-site.xml**:
`  <property>`
`    <name>gluster.daemon.user</name>`
`    <value>yarn</value>`
`  </property>`

Additional For Hadoop 2.x:

**in yarn-site.xml**:

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


**in mapred-site.xml**
`  <property>`
`    <name>mapreduce.jobhistory.done-dir</name>`
`    <value>glusterfs:///mr_history/done</value>`
`  </property>`
  
`  <property>`
`    <name>mapreduce.jobhistory.intermediate-done-dir</name>`
`    <value>glusterfs:///mr_history/tmp</value>`
`  </property>`

### Linux Task Controller 

Setup the LinuxTaskController on each node.  Details can be found here:
[Securing Hadoop](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/ClusterSetup.html#Running_Hadoop_in_Secure_Mode)

**in container-executor.cfg**
`   yarn.nodemanager.linux-container-executor.group=hadoop`
`   banned.users=yarn`
`   min.user.id=1000`
`   allowed.system.users=mapred`

** Set the UID bit on Container Executor **
`  chown root:hadoop $HADOOP_INSTALL/bin/container-executor`
`  chmod 6050 $HADOOP_INSTALL/bin/container-executor`

### Running map/reduce Jobs
Ensure that your Hadoop Cluster users have appropriate permissions to the Gluster Volume.  This can be accomplished with permissions/mode on data files and output directories or ACL.  **Users must have read to data files on the volume and write access to output directories.**  

Once the user has permissions to the required input and output directories, login to the cluster as that user and submit the job.