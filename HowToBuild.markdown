** NOTE: Example below is for Hadoop version 0.20.2 ($GLUSTER_HOME/hdfs/0.20.2) **

* Building the plugin from source [Maven (http://maven.apache.org/) and JDK is required to build the plugin]

  Change to glusterfs-hadoop directory in the GlusterFS source tree and build the plugin.

  # cd $GLUSTER_HOME/hdfs/0.20.2
  # mvn package

  On a successful build the plugin will be present in the `target` directory.
  (NOTE: version number will be a part of the plugin)

  # ls target/
  classes  glusterfs-0.20.2-0.1.jar  maven-archiver  surefire-reports  test-classes

  Copy the plugin to lib/ directory in your $HADOOP_HOME dir.

  # cp target/glusterfs-0.20.2-0.1.jar $HADOOP_HOME/lib

  Copy the sample configuration file that ships with this source (conf/core-site.xml) to conf
  directory in your $HADOOP_HOME dir.

  # cp conf/core-site.xml $HADOOP_HOME/conf
