## Overview.

First, we'll set up hadoop on gluster, with no security, using the yarn container executor (which is the default).

Then we will add the GlusterContainer Executor in as a quick and simple way to run jobs on CDH5 against glusterfs with multitenancy.

## Part 1 : Follow the directions for generic configuration of hadoop on Gluster for CDH5.

[[General_Configuration_CDH5]]

After this, you can execute jobs, but only as the "yarn" user. 

Now, confirm that you can indeed start your yarn service, by exporting environment variables and then starting NM and RM services.  
    
    su yarn
    source env.sh
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start resourcemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start nodemanager

These to services should start without error. 

## Part 2 : Running Hadoop with the GlusterContainerExecutor

1) Add/modify the following properties in the yarn-site.xml file.

    yarn.nodemanager.container-executor.class=        
    org.apache.hadoop.yarn.server.nodemanager.GlusterContainerExecutor   
    yarn.nodemanager.linux-container-executor.group=
    hadoop

2) Create a linux container executor configuration as described at the end of this page.

3) Copy the aforementioned container-executor.cfg file to all machines on your cluster, into the /etc/hadoop/conf/ directory.

4) Now, you must make sure there is some mechanism to ensure that all the **users** and **groups** in the above file have identical UIDs or GIDs.   (i.e. user "hadoop" should have the same uid on both nodes).

4.0) Ensure that users who will execute jobs on the system have >=  the container exectuor  minimum id specified in the file above.

There are many ways to do this do either 4.1,4.2, or 4.3 below:

(4.1) you can copy /etc/passwd and /etc/group from your head node to all others  

(4.2) Follow the IPA based user setup section of  [[ConfiguringHadoop23_SECURE]] OR 

(4.3) Use your companies internal LDAP servers to provision system ids for you.    

--------------------

Restart your yarn and nodemanager services.  To do this, you can follow the snippet in the TESTING STARTUP  section of  [[General_Configuration_CDH5]]

--------------------

Linux Container Executor setup

Create a container-executor.cfg file , and write it out to /etc/hadoop/conf.  You can do this in the shell like so, for a user "tom".  You can add other users as well in a comma separated list (i.e. `allowed.system.users=tom,mary,joe` )

    echo "yarn.nodemanager.linux-container-executor.group=hadoop`
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom" >> /etc/hadoop/conf/ container-executor.cfg
