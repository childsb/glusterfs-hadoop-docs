## Requirements ##

* A GlusterFS 3.3 (or later) volume
* Apache Hadoop 1.x or 2.x
* Java Runtime Environment (JRE) 1.6 +

## Deployment Architecture ##


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

  name:  quick.slave.io
  value: [On/Off], [Yes/No], [1/0]

         NOTE: This option is not tested as of now.

         This is a performance tunable option. Hadoop schedules jobs to hosts that contain the file data part. The job
         then does I/O on the file (via FUSE in case of GlusterFS). When this option is set, the plugin will try to
         do I/O directly from the backed filesystem (ext3, ext4 etc..) the file resides on. Hence read performance
         will improve and job would run faster.


## Usage ##

  Once configured, start Hadoop Map/Reduce daemons

  # cd $HADOOP_HOME
  # ./bin/start-mapred.sh

  If the map/reduce job/task trackers are up, all I/O will be done to GlusterFS.
