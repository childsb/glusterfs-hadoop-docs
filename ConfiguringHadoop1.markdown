**Download and Extract Hadoop**


Navigate to http://hadoop.apache.org/releases.html#Download and download the Apache Hadoop 1.? tar ball to the /opt directory on the JobTracker server.
Extract the tar ball in the /opt directory

**Configure the Plugin**

Edit the $HADOOP_HOME/conf/core-site.xml file to include the following (additional properties can be added)

<INSERT CORE_SITE>

**Modify the Slaves files**

SCP the $HADOOP_HOME directory to each server in the  the cluster, for example:
     scp -r $HADOOP_HOME/ root@svr2:/opt/

**Starting Hadoop**

On the JobTracker server, open a terminal window, navigate to the $HADOOP_HOME directory and start Hadoop by running:
     bin/start-mapred.sh 

**Verifying Hadoop is running successfully on GlusterFS**

In a browser, launch the Hadoop JobTracker UI by navigating to http://<JobTrackerHostName>:50030 
Under the TaskTracker nodes, verify that all the hosts listed within the Hadoop Slaves have reported in. If a host is missing, 
Go to JobTracker, verify that all the TaskTrackers reported in.