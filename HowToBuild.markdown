## Building ##

Building requires a working gluster mount for unit tests.  The unit tests read test resources from glusterconfig.properties - a file which should be present.

* Note: Building the plugin from source [Maven] (http://maven.apache.org/) and JDK  1.6 is required to build the plugin:

1) edit your .bashrc, or else at your terminal run : 

export GLUSTER_VOLUME=MyVolume <-- replace with your preferred volume name (default is HadoopVol)
export GLUSTER_HOST=192.0.1.2 <-- replace with your host (default will be determined at runtime in the JVM)

2) Change to glusterfs-hadoop directory in the GlusterFS source tree and build the plugin by running: 
   mvn package

  On a successful build the plugin (glusterfs-hadoop-<version>) will be present in the `target` directory.

  # ls target/
  classes  glusterfs-hadoop-2.1.2.jar maven-archiver  surefire-reports  test-classes

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