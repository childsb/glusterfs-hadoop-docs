**ERROR**:

While running a Hadoop job, if you see a File Not Found Exception with a "jobtoken does not exist" message

**SOLUTION**: 

This is occurring because the user IDs(UID) and group IDs(GID) are not consistent across the cluster. For example, user "tom" has a UID of 1002 on server1, but on server2 the user "tom" has a UID of 1003. The simplest and recommended approach is to leverage LDAP authentication to resolve this issue. After creating the necessary user and groups on an LDAP server, the servers within the cluster can be configured to use the LDAP server for authentication. A detailed guide on how to set this up for RHEL/CentOS can be found here - https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Deployment_Guide/ch-Configuring_Authentication.html

------------------------------------------------------------
**ERROR**:

Resource glusterfs:/..../.staging/job_1330116323296_0140/job.jar changed on src filesystem (expected 2971811411, was 1330116705875)
 
**SOLUTION**: 

Trying to manually sync up your ntp can fix this: 
service ntpd stop ; sudo ntpdate -s time.nist.gov ; sudo service ntpd start 
 
**NOTES**:

This error occurs because yarn expects the last modified time of a file to be identical on in its FSDownload implementation.  Since file modification times can vary between nodes if the underlying clock is not synchronized in glusterfs, its best to make sure you are tightly regulating your NTP.  In Hadoop 1.x, Mapreduce will not enforce any ntp penalties, but HBase will, so get your clocks straightened out !

------------------------------------------------------------
**ERROR**:
in /var/log/hadoop/mapred/ hadoop-mapred-jobtracker-xxx.log
2013-06-17 22:01:23,820 FATAL org.apache.hadoop.mapred.JobTracker: java.lang.RuntimeException: java.lang.ClassNotFoundException: org.apache.hadoop.fs.glusterfs.GlusterFileSystem
 
**SOLUTION**: Copy gluster jar to /usr/lib/hadoop/lib from /usr/share/java/
--------------------------------
**ERROR**:
notice: /Stage[2]/Hdp-ganglia::Monitor/Hdp::Package[ganglia-monitor]/Hdp::Package::Process_pkg[ganglia-monitor]/Anchor[hdp::package::ganglia-monitor::end]: Dependency Package[ganglia-gmond-3.2.0] has failures: true
warning: /Stage[2]/Hdp-ganglia::Monitor/Hdp::Package[ganglia-monitor]/Hdp::Package::Process_pkg[ganglia-monitor]/Anchor[hdp::package::ganglia-monitor::end]: Skipping because of failed dependencies
notice: /Stage[2]/Hdp-ganglia::Config/Anchor[hdp-ganglia::config::begin]: Dependency Package[ganglia-gmond-3.2.0] has failures: true
warning: /Stage[2]/Hdp-ganglia::Config/Anchor[hdp-ganglia::config::begin]: Skipping because of failed dependencies
 
**SOLUTION**:
wget http://public-repo-1.hortonworks.com/ambari/centos6/1.x/GA/ambari.repo
 
cp ambari.repo /etc/yum.repos.d
 
---------------------------------------------
**ERROR**:
2013-07-24 21:56:21,386 INFO org.apache.hadoop.fs.FileSystem: Gluster Output Buffering size configured to 0 bytes.
2013-07-24 21:56:21,388 INFO org.apache.hadoop.mapred.JobTracker: Setting safe mode to false. Requested by : mapred
2013-07-24 21:56:21,392 INFO org.apache.hadoop.util.NativeCodeLoader: Loaded the native-hadoop library
2013-07-24 21:56:21,395 INFO org.apache.hadoop.mapred.JobTracker: Cleaning up the system directory
2013-07-24 21:56:21,418 FATAL org.apache.hadoop.mapred.JobTracker: java.lang.RuntimeException: org.apache.hadoop.util.Shell$ExitCchmod -R 1777 /mnt/glusterfsodeException: chmod: cannot access `/mnt/glusterfs/mapred': No such file or directory
 
        at org.apache.hadoop.fs.glusterfs.GlusterFileSystem.setPermission(GlusterFileSystem.java:383)
        at org.apache.hadoop.fs.glusterfs.GlusterFileSystem.mkdirs(GlusterFileSystem.java:205)
        at org.apache.hadoop.fs.FileSystem.mkdirs(FileSystem.java:1166)
 
**SOLUTION**:
chmod -R 1777 /mnt/glusterfs
RESTART MAPRED
 
------------------------------------------------------------
**ERROR**:
notice: /Stage[2]/Hdp-nagios::Server::Packages/Exec[remove_package nagios]/returns: executed successfully
err: /Stage[2]/Hdp-nagios::Server::Packages/Hdp::Package[nagios-plugins]/Hdp::Package::Process_pkg[nagios-plugins]/Package[nagios-plugins-1.4.9]/ensure: change from absent to present failed: Execution of '/usr/bin/yum -d 0 -e 0 -y install nagios-plugins-1.4.9' returned 1: Error: Package: nagios-plugins-1.4.9-1.x86_64 (HDP-UTILS-1.1.0.15)
           Requires: perl(Net::SNMP)
You could try using --skip-broken to work around the problem
You could try running: rpm -Va --nofiles --nodigest
 
**SOLUTION**: yum install epel-release
 
------------------------------------------------------------
 
**ERROR**:
root@gprfs001 hadoop-1.2.0.1.3.2.0-110]# bin/hadoop jar hadoop-test-1.2.0.1.3.2.0-110.jar TestDFSIO -write -nrFiles 10 -fileSize 100
TestDFSIO.0.0.4
13/10/07 11:18:37 INFO fs.TestDFSIO: nrFiles = 10
13/10/07 11:18:37 INFO fs.TestDFSIO: fileSize (MB) = 100
13/10/07 11:18:37 INFO fs.TestDFSIO: bufferSize = 1000000
13/10/07 11:18:37 INFO glusterfs.GlusterFileSystemCRC: Initializing gluster volume..
13/10/07 11:18:37 INFO glusterfs.GlusterFileSystem: Configuring GlusterFS
13/10/07 11:18:37 INFO glusterfs.GlusterFileSystem: Initializing GlusterFS,  CRC disabled.
13/10/07 11:18:37 INFO glusterfs.GlusterFileSystem: Configuring GlusterFS
13/10/07 11:18:37 INFO glusterfs.GlusterFileSystemCRC: Initializing gluster volume..
13/10/07 11:18:37 INFO fs.TestDFSIO: creating control file: 100 mega bytes, 10 files
13/10/07 11:18:37 INFO util.NativeCodeLoader: Loaded the native-hadoop library
13/10/07 11:18:38 INFO fs.TestDFSIO: created control files for: 10 files
13/10/07 11:18:38 INFO mapred.FileInputFormat: Total input paths to process : 10
13/10/07 11:18:38 INFO mapred.JobClient: Cleaning up the staging area glusterfs:/tmp/hadoop/mapred/staging/root/.staging/job_201310071111_0031
13/10/07 11:18:38 ERROR security.UserGroupInformation: PriviledgedActionException as:root cause:java.io.IOException: Cannot get layout
java.io.IOException: Cannot get layout
at org.apache.hadoop.fs.glusterfs.GlusterFSXattr.execGetFattr(GlusterFSXattr.java:225)
at org.apache.hadoop.fs.glusterfs.GlusterFSXattr.getPathInfo(GlusterFSXattr.java:84)
at org.apache.hadoop.fs.glusterfs.GlusterVolume.getFileBlockLocations(GlusterVolume.java:155)
at org.apache.hadoop.fs.FilterFileSystem.getFileBlockLocations(FilterFileSystem.java:98)
at org.apache.hadoop.mapred.FileInputFormat.getSplits(FileInputFormat.java:231)
at org.apache.hadoop.mapred.JobClient.writeOldSplits(JobClient.java:1081)
at org.apache.hadoop.mapred.JobClient.writeSplits(JobClient.java:1073)
at org.apache.hadoop.mapred.JobClient.access$700(JobClient.java:179)
at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:983)
at org.apache.hadoop.mapred.JobClient$2.run(JobClient.java:936)
at java.security.AccessController.doPrivileged(Native Method)
 
 
**POSSIBLE SOLUTION(S)**:
1) gluster volume is not mounted (either because the mount failed or you forgot to mount it)
    Mount the gluster volume.
2) If you get this error when  running the hadoop with RHS command from a remote machine
     using ssh your problem could be the following /etc/sudoers setting. 
     Note: getfattr uses sudo, when you run hadoop by passing the hadoop command in the ssh command line
     you don't get a tty, so you must comment out the following line in /etc/sudoers.
     #Defaults    requiretty


------------------------------------------------------------
 ** ERROR ** : 
gluster peer probe <hostname>
peer probe: failed: Probe returned with unknown errno 107

/var/log/glusterfs/cli.log:
[2013-12-12 04:10:34.603003] E [cli-rpc-ops.c:116:gf_cli_probe_cbk] 0-cli: Probe returned with unknown errno 107
[2013-12-12 04:10:34.603096] I [input.c:36:cli_batch] 0-: Exiting with: -1
[2013-12-12 16:08:24.432419] I [socket.c:3485:socket_init] 0-glusterfs: SSL support is NOT enabled
[2013-12-12 16:08:24.432481] I [socket.c:3500:socket_init] 0-glusterfs: using system polling thread
[2013-12-12 16:08:24.460613] I [socket.c:3485:socket_init] 0-glusterfs: SSL support is NOT enabled
[2013-12-12 16:08:24.460655] I [socket.c:3500:socket_init] 0-glusterfs: using system polling thread
[2013-12-12 16:08:24.796256] I [socket.c:2235:socket_event_handler] 0-transport: disconnecting now
[2013-12-12 16:08:24.829402] I [cli-rpc-ops.c:111:gf_cli_probe_cbk] 0-cli: Received resp to probe
[2013-12-12 16:08:24.829454] E [cli-rpc-ops.c:116:gf_cli_probe_cbk] 0-cli: Probe returned with unknown errno 107
[2013-12-12 16:08:24.829531] I [input.c:36:cli_batch] 0-: Exiting with: -1


** SOLUTIONS ** 
You might be getting this error because linux enables iptables by default.

option 1) 
use iptables -I to add rules to allow gluster to access ports it needs with
protocols it needs. 

option 2) 
turn off the firewall completely.  this is not the best option in most cases.
iptables -F  

Notes:
When adding hosts to a gluster trusted storage pool, using "gluster peer probe" you may encounter this error.  Out-of-the-box Linux has security enabled, you have to disable it if you don't want it, or modify it to meet your needs. The firewall (iptables) is part of default security, selinux is the other part.
