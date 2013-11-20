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
**In mapred-site.xml**:
`  <property>`
`   <name>mapreduce.jobtracker.staging.root.dir</name>`
`   <value>glusterfs:///job-staging</value> `
`   </property>`


Additional For Hadoop 2.x:

**in yarn-site.xml**:
`  <property>`
`   <name>yarn.app.mapreduce.am.staging-dir</name>    `
`   <value>glusterfs:///job-staging-yarn</value> `
`   </property>`

**in mapred-site.xml**
`  <property>`
`    <name>mapreduce.jobhistory.done-dir</name>`
`    <value>glusterfs:///job-history/done</value>`
`  </property>`
  
`  <property>`
`    <name>mapreduce.jobhistory.intermediate-done-dir</name>`
`    <value>glusterfs:///job-history/intermediate-done</value>`
`  </property>`

** One last step: Make your user's working directory writable by your mapreduce daemons ** 

Now, if you want bob to submit jobs to the cluster, and allow "yarn" to run them, you can set your acl's for all files created under bob's home directory: 

setfacl -dm u::rwx,g::rwx,o::r /mnt/glusterfs/user/bob/

This assumes that "bob" and "yarn" are in the same group.

At the end of mapreduce jobs, files are often generally placed in directories on the DFS corresponding to the home directory where the job was originally run.  This is because the job client creates paths from the default working directory.   For example, jobs such as "calculate pi" create output directories by using relative paths, such as this:

` static private final Path TMP_DIR = new Path(MyJobClass.class.getSimpleName() + System.currentTimeInMillis());

As you can see above, a path is created which is NOT qualified.  Thus, if user "bob" runs the calculate pi job, the output will be in a path (on the DFS) such as:

"/user/bob/MyJobClass+1384972545371"

### Running map/reduce Jobs
Ensure that your Hadoop Cluster users have appropriate permissions to the Gluster Volume.  This can be accomplished with permissions/mode on data files and output directories or ACL.  **Users must have read to data files on the volume and write access to output directories.**  

Once the user has permissions to the required input and output directories, login to the cluster as that user and submit the job.