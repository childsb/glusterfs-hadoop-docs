## Pre-Requisites ##

- Building requires a working gluster mount for unit tests.  
-  [Maven] (http://maven.apache.org/) and JDK  1.6 are required to build the plugin

## Build Process ##

1) edit your .bashrc, or else at your terminal run : 

export GLUSTER_VOLUME=MyVolume <-- replace with your preferred volume name (default is HadoopVol)
export GLUSTER_HOST=192.0.1.2 <-- replace with your host (default will be determined at runtime in the JVM)

2) Change to glusterfs-hadoop directory in the GlusterFS source tree and build the plugin by running: 
   mvn package

  On a successful build the plugin (glusterfs-hadoop-<version>) will be present in the `target` directory.

  # ls target/
  classes  glusterfs-hadoop-2.1.2.jar maven-archiver  surefire-reports  test-classes