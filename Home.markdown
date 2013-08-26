## Introduction ##

While Apache Hadoop typically ships with the Hadoop Distributed FileSystem (HDFS), [Hadoop can be configured to use alternate FileSystems] (http://wiki.apache.org/hadoop/HCFS) by leveraging its inherent pluggable filesystem architecture. One achieves this by building a plugin that acts as a mediator between the Hadoop FileSystem Interface and the desired FileSystem, and configuring Hadoop to use the plugin. This project contains the plugin required to enable GlusterFS to be used as a Hadoop FileSystem. 

## Releases ##

The Plugin supports both Apache Hadoop 1.x and 2.x releases and has thus far been successfully tested against both the Hortonworks and Intel Hadoop Distributions. The plugin is backwards compatible and thus we recommend you always obtain the latest version (the highest numerical value).

[Plugin releases are available on our Archiva repo](http://23.23.239.119/archiva/browse/org.apache.hadoop.fs.glusterfs/glusterfs-hadoop)

## Configuration ##

Our [[Configuration]] page provides a guide on how to configure GlusterFS and Hadoop to make use of the plugin.

## How To Contribute ##

Our [[HowToContribute]] page contains information on how to participate in the development of the plugin.



## For Hackers ##

* Source Layout (./src/)

org.apache.hadoop.fs.glusters/GlusterFSBrickClass.java
org.apache.hadoop.fs.glusters/GlusterFSXattr.java            <--- Fetch/Parse Extended Attributes of a file
org.apache.hadoop.fs.glusters/GlusterFUSEInputStream.java    <--- Input Stream (instantiated during open() calls; quick read from backed FS)
org.apache.hadoop.fs.glusters/GlusterFSBrickRepl.java
org.apache.hadoop.fs.glusters/GlusterFUSEOutputStream.java   <--- Output Stream (instantiated during creat() calls)
org.apache.hadoop.fs.glusters/GlusterFileSystem.java         <--- Entry Point for the plugin (extends Hadoop FileSystem class)

org.gluster.test.AppTest.java                  <--- Your test cases go here (if any :-))

./tools/build-deploy-jar.py                                                  <--- Build and Deployment Script
./conf/core-site.xml                                                         <--- Sample configuration file
./pom.xml                                                                    <--- build XML file (used by maven)

./COPYING                                                                    <--- License
./README                                                                     <--- This file



## Jenkins ##

  At the moment, you need to run as root - this can be done by modifying this line in the jenkins init.d/ script.
  This is because of the mount command issued in the GlusterFileSystem. 
  
  #Method 1) Modify JENKINS_USER in /etc/sysconfig/jenkins
  JENKINS_USER=root

  #Method 2) Directly modify /etc/init.d/jenkins 
  #daemon --user "$JENKINS_USER" --pidfile "$JENKINS_PID_FILE" $JAVA_CMD $PARAMS > /dev/null
  echo "WARNING: RUNNING AS ROOT" 
  daemon --user root --pidfile "$JENKINS_PID_FILE" $JAVA_CMD $PARAMS > /dev/null

## Building ##

Building requires a working gluster mount for unit tests. 
The unit tests read test resources from glusterconfig.properties - a file which should be present 

1) edit your .bashrc, or else at your terminal run : 

export GLUSTER_VOLUME=MyVolume <-- replace with your preferred volume name (default is HadoopVol)
export GLUSTER_HOST=192.0.1.2 <-- replace with your host (default will be determined at runtime in the JVM)

2) run: 
   mvn package