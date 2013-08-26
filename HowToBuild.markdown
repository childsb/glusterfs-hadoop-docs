# Basic Build #

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

# Distributed Build #

 In case it is tedious to do the above steps(s) on all hosts in the cluster; use the build-and-deploy.py script to build the plugin in one place and deploy it (along with the configuration file on all other hosts).

  This should be run on the host which is that hadoop master [Job Tracker].

* STEPS (You would have done Step 1 and 2 anyway while deploying Hadoop)

  1. Edit conf/slaves file in your hadoop distribution; one line for each slave.
  2. Setup password-less ssh b/w hadoop master and slave(s).
  3. Edit conf/core-site.xml with all glusterfs related configurations (see CONFIGURATION)
  4. Run the following
     # cd $GLUSTER_HOME/hdfs/0.20.2/tools
     # python ./build-and-deploy.py -b -d /path/to/hadoop/home -c

     This will build the plugin and copy it (and the config file) to all slaves (mentioned in $HADOOP_HOME/conf/slaves).

   Script options:
     -b : build the plugin
     -d : location of hadoop directory
     -c : deploy core-site.xml
     -m : deploy mapred-site.xml
     -h : deploy hadoop-env.sh