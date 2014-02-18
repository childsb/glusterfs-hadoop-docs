Apache Oozie on GlusterFS-Hadoop
--------------------------------

** Please note that our support of oozie is still somewhat experimental.** 

This wiki page demonstrates how to run oozie on top of gluster by utilization of the glusterfs-hadoop plugin.  It can also be used as a generic proof-of-concept of how to run oozie on any HCFS file system (this is a somehwat underdocumented feature of oozie, see https://issues.apache.org/jira/browse/OOZIE-1695 for progress we are making on this front).

In this setup, we will run a job as user "oozie", although that is quite unconventional.  We are actively working to verify more conventional multiuser workloads. 

First, setup your hadoop cluster and confirm that you can run a standard mapreduce job as a user, such as tom.  See ConfiguringAmbari2 for details.

--------------------

** Now, add oozie to the impersonator settings in your core-site.xml. **  These two parameters are used in the YARN MapReduce job submission process.  If they are not permissive enough, oozie will fail with the error "oozie is not allowed to impersonate ...", when it attempts to submit jobs.   

    <!-- machine addresses which allow oozie to impersonate --> 
    <property>
    <name>hadoop.proxyuser.oozie.hosts</name>
    <value>hadoopclient.lab.redhat.com</value>
    </property>
    
    <!-- * will allow oozie to impersonate any hadoop group for submitting jobs -->
    <property>
    <name>hadoop.proxyuser.oozie.groups</name>
    <value>*</value>
    </property>

--------------------
** Setup your task controllers to support oozie, ** and make sure oozie is above the min user id.  You may have to take the below and adapt it to your already existing task controller file. 

    cat << EOF > execfile
    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=200
    allowed.system.users=tom,oozie,sally,hadoop
    EOF
-------------------

** Make sure the file above is on all nodes of your cluster, and is identical.**  You will want to copy it over like so:

    scp execfile node1:/etc/hadoop/conf/container-executor.cfg
    scp execfile node2:/etc/hadoop/conf/container-executor.cfg
---------------------

** Now, make sure that oozie allows the gluster file system.  ** 
You do this by adding this to your oozie-site.xml. 

    <property>  
  
<name>oozie.service.HadoopAccessorService.supported.filesystems</name>
    <value>glusterfs</value>
    </property>

** Drop the latest version of the glusterfs-hadoop plugin into libext, so that oozie runtime sees it** and can use it to read your workflows.   

    ln -s /usr/lib/hadoop/lib/glusterfs-hadoop-2.1.6.jar /usr/lib/oozie/libext/glusterfs-hadoop-symlink.jar

Restart all your YARN services, and start oozie.

-------------------------

To be safe, since ** its not always consistent how oozie deployments are adding jars to the oozie web app ** , you can also add the plugin to the running oozie war application library path.

    ln -s /usr/lib/hadoop/lib/glusterfs-hadoop-2.1.6.jar /usr/lib/oozie/libext/glusterfs-hadoop-symlink.jar

---------------------------

Okay ! You did it.  Now, we will breifly overview how to submit jobs into an oozie cluster.  

Before moving on, breifly scan /var/log/oozie/oozie.log for errors, and if anything particularly vexing comes up, feel free to ping us on the gluster mailing list.

---------------------------
** How to write your own jobs ** 

** For an example of a full workflow ** 

You can clone and run the oozie-smokes run.sh script, found here in the "oozie-smokes" directory, https://forge.gluster.org:hadoop/simple-smokes.git.  That project has a "run.sh" script which you can use to run and debug job submission (but note that it might report job failure, even though the job succeeds --- see https://issues.apache.org/jira/browse/OOZIE-1700 with accompanying video of this oozie oddity http://www.youtube.com/watch?v=EPdkJ3zMp0c&feature=em-upload_owner).

** Hers how to create your own oozie workflow that runs on gluster from scratch ** 

Each oozie workflow consists of 3 components:
* properties file (stored **LOCALLY**, or parameters sent at runtime )
* workflow.xml file (this is always in the **DFS** )
* libraries directory (this is also stored in the **DFS**)

We will walk through creation of those resources below.

Create a job.properties file like the one below: (replace 8050 with the value of the port portion of "yarn.resourcemanager.address").  In our example, we call it j2.properties, but you can **call this file anything you want**.

    nameNode=glusterfs:///
    jobTracker=yarn.server.node:8050
    queueName=default
    oozie.wf.application.path=glusterfs:///user/oozie/oozietest2/

Now, create your workflow.xml file, along with any libraries, into the dfs specified path in the last parameter above.  An example workflow file is here: https://forge.gluster.org/hadoop/simple-smokes/blobs/master/oozie-smoke/workflow.xml.  **YOU MUST name this file workflow.xml**.  Oozie will look to it when you submit your job, as the definition of what to run.

    hadoop fs -put workflow.xml glusterfs:///user/oozie/oozietest2/workflow.xml
    
**  Time to create the last component: you will need one of more binaries in that oozie will run. **   

Binaries are simply the jar file which are the code which oozie calls in tasks.  ** Oozie can call "main" methods in arbitrary java classes, or it can use its MapReduce XML definitions to call Mapper and Reducer classes. **  It can also run pig/hive tasks as well. In any case, one way or another certain jars must obviously be made available to the job submission runtime.   In this particular case, we are using a MapReduce example.

This is done by putting them in the "lib/" folder under your job's main directory.   

** The simplest place to start is with the oozie examples jar,** like this one: https://forge.gluster.org/hadoop/simple-smokes/blobs/master/oozie-smoke/lib/oozie-examples-2.3.2-cdh3u4.jar.  So, the final step in creating our "oozietest2" job, might be to do this:
    
    wget https://forge.gluster.org/hadoop/simple-smokes/blobs/master/oozie-smoke/lib/oozie-examples-2.3.2-cdh3u4.jar -O myjar.jar
    hadoop fs -put myjar.jar glusterfs:///user/oozie/oozietest2/lib/myjar.jar
    
** At this point, you have created all the necessary components for your workflow, ** both locally (your job.properties file) and on the DFS (your DFS home job directory has a workflow.xml and jar files under lib/)

Now, finally, you can **  submit your oozie job **  like this: 

    oozie job -oozie http://localhost:11000/oozie -config j2.properties -run
    
** And monitor its progress in the YARN UI **  (you can also monitor it using oozie job -log commands, but sometimes it appears that OOZIE-1700 causes oozie to report failure when jobs actually succeed, so for now, we suggest monitoring the actual YARN history server).


