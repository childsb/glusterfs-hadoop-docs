## Pre-Requisites ##

- You *should* have a working gluster mount for unit tests, although they may feasibly pass without a gluster mount since the proxy writes into a local file system, at least for versions 1.x and 2.x of the plugin.

- [Maven] (supported version 3.1, the build is known to fail on older maven versions which seem not to support @Ignore annotations).  (http://maven.apache.org/).

- JDK  1.6+.  OpenJDK is fine for development.   Different deployments may use different vendor specific JVM's, however, so you should always test your MapReduce code according to whats on your cluster.

## Build Process ##

1) edit your .bashrc, or else at your terminal run : 

export GLUSTER_VOLUME=MyVolume <-- replace with your preferred volume name (default is HadoopVol)
export GLUSTER_HOST=192.0.1.2 <-- replace with your host (default will be determined at runtime in the JVM)

2) Change to glusterfs-hadoop directory in the GlusterFS source tree and build the plugin by running: 
   mvn package

  On a successful build the plugin (glusterfs-hadoop-<version>) will be present in the `target` directory.

  # ls target/
  classes  glusterfs-hadoop-2.1.2.jar maven-archiver  surefire-reports  test-classes