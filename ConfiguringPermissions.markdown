The following steps provide a guide on how to configure Hadoop to run under the "mapred" user account. 

**Assumptions:**

*  the Hadoop tarball has been extracted to /opt/hadoop-1.2.1.
*  mapred.local.dir is set to /mnt/brick1/mapredlocal within conf/mapred-site.xml
*  on each server, the brick has been mounted as /mnt/brick1

**Create the User and Group**
Create group "hadoop". Create user "mapred". Add user "mapred" to group "hadoop"

** Set the permissions for Hadoop **
chmod 777 /opt/hadoop-1.2.1/*.jar

mkdir -p /mnt/brick1/mapredlocal
chown -R mapred:hadoop /mnt/brick1/mapredlocal
chmod -R 755 /mnt/brick1/mapredlocal

chmod a+x /opt/hadoop-1.2.1/conf/
chown -R mapred:hadoop /opt/hadoop-1.2.1/conf/../
chmod -R 755 /opt/hadoop-1.2.1/conf/../

** Set the permissions for getfattr**
vi /etc/sudoers.d/gluster
mapred ALL= NOPASSWD: /usr/bin/getfattr
chmod 440 /etc/sudoers.d/gluster

** Set the permissions for the gluster mount **
chmod -R 1777 /mnt/glusterfs
su mapred
mkdir -p /mnt/glusterfs/mapred/system

** Start the JobTracker and TaskTracker under the mapred user **
su mapred
bin/hadoop-daemon.sh --config /opt/hadoop-1.2.1/conf/ start jobtracker
bin/hadoop-daemon.sh --config /opt/hadoop-1.2.1/conf/ start tasktracker