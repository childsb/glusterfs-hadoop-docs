# Securing your Hadoop Cluster
Hadoop can run in multiple configurations with varying security.  Each section below outlines additional setup configuration

## Run-as-root
This is the least secure method.  All services are started as root, and map reduce jobs are launched by the 'root' user.  This is insecure so avoid this.

## Single User
The single tenant non-root setup is covered by the basic Install Guide.

## Multi-User
In order for hadoop to run in full multi-user mode, a special user has to be designated to run the hadoop daemons.  This user may be restricted from logging in, or accessing the entire file system.  However, the user must be granted read and write permission to the map/reduce staging directory.  This is accomplished using POSIX ACLs.  To enable ACLs on your gluster volume, mount it with the ACL flag (on every node in the cluster).

### fstab entry for Gluster Volume w/ACL
`localhost:/gv0 /mnt/glusterfs glusterfs acl,auto,transport=tcp 0 0` 

### Create the Hadoop Daemon User
`adduser --no-create-home --system --uid 1357 -gid 1357 jobtracker`

### Create Hadoop Users
Hadoop users can be created like other users (LDAP, command line, etc) and require no special configuration

### Specify the Hadoop Daemon User
The GlusterFS plugin will add the hadoop daemon user to certain system directories to allow access across the cluster.  It's important that the staging directory and the Hadoop Daemon user are supplied in configuration:

**In core-site.xml**:
`  <property>`
`    <name>gluster.daemon.user</name>`
`    <value>jobtracker</value>`
`  </property>`
**In mapred-site.xml**:
`  <property>`
`   <name>mapreduce.jobtracker.staging.root.dir</name>`
`   <value>glusterfs:///jobtracker-staging</value> `
`   </property>`

### Running map/reduce Jobs
Ensure that your Hadoop Cluster users have appropriate permissions to the Gluster Volume.  This can be accomplished with permissions/mode on data files and output directories or ACL.  Users must have read to data files on the volume and write access to output directories.  Then simply login to the system as that user and launch the map/reduce job.

