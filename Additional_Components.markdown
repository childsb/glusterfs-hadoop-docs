#Using HBase#

**Optional: Vagrant setup for 2 node test cluster**

If you are just testing the plugin and want to experiment with Hbase, we have a vagrant-gluster-hbase on fedora19 setup which can be run to automatically provision a 2 node gluster and hbase cluster with smoke tests: 

    git clone https://forge.gluster.org/vagrant/fedora19-gluster/ 
    cd gluster-hbase-example
    vagrant up

##Setting Up

For HBase to be running, you really will need to enable two services: Zookeeper and HBase.  Zookeeper can be started for you by HBase, if you configure it to do so.  In any case, you need a zookeeper service to elect and maintain masters, region servers to store your and shard your data, and an HMaster to coordinate the table information and monitor region servers.  Most important of all is to note that your /etc/hosts has to be perfect for everything to work properly, based on the fragile nature of how HBase connects to hosts. 

So , first make sure you understand how hbase works before following this tutorial:

http://hbase.apache.org/book/master.html

http://hbase.apache.org/book/zookeeper.html

http://hbase.apache.org/book/regionserver.arch.html


Now: to set up, we assume that java , glusterfs, attr, and psmisc are installed, and that there is a gluster volume mounted on /mnt/glusterfs, and that each region server in your cluster has that volume mounted:

- wget hbase from your preferred apache mirror as a tarball

- setup passwordless ssh from your master node to intended region server nodes 

- disable security by turning off iptables, firewalld.service , and setenforce 0

- untar hbase on each node 

-    wget  the glusterfs hadoop plugin from http://rhbd.s3.amazonaws.com/maven/indexV2.html, and put it in /usr/lib/hbase/lib/

- on each node, update your hbase-env.sh script to point to your java installation:

    export JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk.x86_64"

##Setting configuration properties

Set the following properties: 


    hbase.zookeeper.property.clientPort = (zookeeper properties file)
    hbase.cluster.distributed = true
    hbase.zookeeper.quorum = (select 1,3,5... zk machines)
    hbase.zookeeper.property.dataDir = (data dir property in /usr/lib/zookeeper...)


##Startup

You can start up a region server and an hbase master easily like this 

`
./bin/hbase-daemon.sh start zookeeper 
./bin/hbase-daemon.sh start master 
./bin/hbase-daemon.sh start regionserver 
`

After this, you can (on other servers), ssh in and follow the same steps, to start regionservers.  These regionserver daemons will be started when you start hbase.  Make sure their configuration matches that of the master.

##Testing

- run the following smoke test to confirm operation

    sudo bin/hbase shell -d <<EOF
    create 't1','f1' 
    put 't1', 'row1', 'f1:a', 'val1'
    scan 't1'
    EOF


##Troubleshooting

1) If you get zookeeper or HMaster is down exceptions, make sure your /etc/hosts file should look something like this: 

    127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
    99.10.10.11	rs1
    99.10.10.12	hmaster

(note that in the above, we have localhost = 127.0.0.1, and that there is a mapping both for 127.* and for the external IP 99.10.10.11).  
 
-----------------------------------------------------
# Using Impala #

Note that currently, Impala doesn't yet support GlusterFS.  But you can set up the GlusterFileSystem to be the backing store for impala, and if you recompile bits of impala, you can get it to work, in theory.  

*For those really interested in running impala on alternate file systems, contact us directly and we are happy to discuss the details*

We have indeed managed to glue it in to CDH5, but unless a comprehensive set of changes is made to Impala (as of July 15, 2014), you cannot run it on non-HDFS file systems.


For the brave, however, here are our notes on configuring Impala.  You can actually get Impala to start up and read from the gluster filesystem, and even use impyla to create an impala table, using approximately the directions below.

Once we work to make Impala allow any HCFS implementation (Should be removing a few possibly unnecessary casting checks for HDFS), Impala should theoretically support any File system implementation as per the protocol below .

To set up impala , you first must set up hive in server mode.  That can be done following the directions in these two places:

[Installing Hive](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hive_install.html)
[Configuring the Hive Metastore](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hive_metastore_configure.html)

In the process, you will set up HiveServer2.

Note however that impala creates its own tables.  So, even though you will want to set up hive - tables in hive won't necessarily be immediately queriable by impala.

If you are on fedora, mysql is actually packaged as "MariaDB", and works as a replacement for MySQL server for the metastore.

Make sure to Set up a non-embdeed (i.e. mysql or postgre) metastore, as Impala will not work with a derby metastore.

At that point, you can do the following.

1) Edit the /etc/init.d/impala-server script like so:

Add this variable.

    D2="${IMPALA_SERVER_ARGS} --abort_on_config_error=false"
    and modify this line /bin/su -s /bin/bash -c "/bin/bash -c 'cd ${RUNDIR} && echo \$\$ > ${PIDFILE} && exec ${EXEC_PATH} ${IMPALA_SERVER_ARGS}

... to look like this...

    /bin/su -s /bin/bash -c "/bin/bash -c 'cd ${RUNDIR} && echo \$\$ > ${PIDFILE} && 
exec ${EXEC_PATH} ${D2}

 

2) Now put the glusterfs-hadoop jar file in /var/lib/impala/.  This is a critical step.  The impala server copies these

files into the classpath on startup.  If the plugin is not there - then impala won't be able to see the gluster jars. 

3) Now, make sure your HiveServer is stopped.  Update the hive-site.xml to contain the usual glusterfs parameters.  This is important because Impala will use HiveServer to do gluster based queries : If HiveServer doesn't know how to load the plugin, it wont work.  

4) Now, start impala services:

    sudo service impala-server restart ; 
    sudo service impala-state-store restart ; 
    sudo service impala-catalog restart;

And confirm that they are running:

    netstat -tupln | grep impalad

The above command should list several services in the 21000-213000 range.

The imapad service running on port 21050 is the one we will connect to using the python impyla client.

 

5) And install the impyla python client.  This client is more robust than the impala-shell app with respect

to platform specificities.  Lets put some data into the dfs first:

 
    hadoop fs -mkdir /tmp/t1/
    hadoop fs -copyFromLocal /etc/passwd /tmp/t1/
    yum install python-pip
    easy_install -U setup_tools
    pip install impyla
    from impala.dbapi import connect
    conn = connect(host='my.host.com', port=21050)
    cursor = conn.cursor()
    cursor.execute('create external table t1 (d string) location \'/tmp/t1\'')
    cursor.execute('SELECT * FROM t1 LIMIT 100')
    print cursor.description # prints the result set's schema
    results = cursor.fetchall()

Theoretically (pending allowance of any HCFS implementation) the above should work for Impala on glusterfs.   Currently, however, we're finding that this workflow results in a runtime exception while casting to HDFS.

--------------------


# Using Oozie #


** Please note that our support of oozie is still somewhat experimental.** 

This wiki page demonstrates how to run oozie on top of gluster by utilization of the glusterfs-hadoop plugin.  It can also be used as a generic proof-of-concept of how to run oozie on any HCFS file system (this is a somehwat underdocumented feature of oozie, see https://issues.apache.org/jira/browse/OOZIE-1695 for progress we are making on this front).

In this setup, we will run a job as user "oozie", although that is quite unconventional.  We are actively working to verify more conventional multiuser workloads. 

First, setup your hadoop cluster and confirm that you can run a standard mapreduce job as a user, such as tom.  See ConfiguringAmbari2 for details.

**Now, add oozie to the impersonator settings in your core-site.xml.**  These two parameters are used in the YARN MapReduce job submission process.  If they are not permissive enough, oozie will fail with the error "oozie is not allowed to impersonate ...", when it attempts to submit jobs.   

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


**Setup your task controllers to support oozie,** and make sure oozie is above the min user id.  You may have to take the below and adapt it to your already existing task controller file. 

    cat << EOF > execfile
    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=200
    allowed.system.users=tom,oozie,sally,hadoop
    EOF


**Make sure the file above is on all nodes of your cluster, and is identical.**  You will want to copy it over like so:

    scp execfile node1:/etc/hadoop/conf/container-executor.cfg
    scp execfile node2:/etc/hadoop/conf/container-executor.cfg


**Now, make sure that oozie allows the gluster file system.** 
You do this by adding this to your oozie-site.xml. 

    <property>
    <name>oozie.service.HadoopAccessorService.supported.filesystems</name>
    <value>glusterfs</value>
    </property>



**Drop the latest version of the glusterfs-hadoop plugin into libext, so that oozie runtime sees it** and can use it to read your workflows.   

    ln -s /usr/lib/hadoop/lib/glusterfs-hadoop-2.1.6.jar /usr/lib/oozie/libext/glusterfs-hadoop-symlink.jar

Restart all your YARN services, and start oozie.

-------------------------

To be safe, since **its not always consistent how oozie deployments are adding jars to the oozie web app** , you can also add the plugin to the running oozie war application library path.

    ln -s /usr/lib/hadoop/lib/glusterfs-hadoop-2.1.6.jar /usr/lib/oozie/libext/glusterfs-hadoop-symlink.jar

---------------------------

Okay ! You did it.  Now, we will breifly overview how to submit jobs into an oozie cluster.  

Before moving on, breifly scan /var/log/oozie/oozie.log for errors, and if anything particularly vexing comes up, feel free to ping us on the gluster mailing list.

---------------------------
**How to write your own jobs** 

*For an example of a full workflow*

You can clone and run the oozie-smokes run.sh script, found here in the "oozie-smokes" directory, https://forge.gluster.org:hadoop/simple-smokes.git.  That project has a "run.sh" script which you can use to run and debug job submission (but note that it might report job failure, even though the job succeeds --- see https://issues.apache.org/jira/browse/OOZIE-1700 with accompanying video of this oozie oddity http://www.youtube.com/watch?v=EPdkJ3zMp0c&feature=em-upload_owner).

**Hers how to create your own oozie workflow that runs on gluster from scratch** 

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
    
**Time to create the last component: you will need one of more binaries in that oozie will run.**   

Binaries are simply the jar file which are the code which oozie calls in tasks.  **Oozie can call "main" methods in arbitrary java classes, or it can use its MapReduce XML definitions to call Mapper and Reducer classes.**  It can also run pig/hive tasks as well. In any case, one way or another certain jars must obviously be made available to the job submission runtime.   In this particular case, we are using a MapReduce example.

This is done by putting them in the "lib/" folder under your job's main directory.   

**The simplest place to start is with the oozie examples jar,** like this one: https://forge.gluster.org/hadoop/simple-smokes/blobs/master/oozie-smoke/lib/oozie-examples-2.3.2-cdh3u4.jar.  So, the final step in creating our "oozietest2" job, might be to do this:
    
    wget https://forge.gluster.org/hadoop/simple-smokes/blobs/master/oozie-smoke/lib/oozie-examples-2.3.2-cdh3u4.jar -O myjar.jar
    hadoop fs -put myjar.jar glusterfs:///user/oozie/oozietest2/lib/myjar.jar
    
**At this point, you have created all the necessary components for your workflow,** both locally (your job.properties file) and on the DFS (your DFS home job directory has a workflow.xml and jar files under lib/)

Now, finally, you can **submit your oozie job**  like this: 

    oozie job -oozie http://localhost:11000/oozie -config j2.properties -run
    
**And monitor its progress in the YARN UI**  (you can also monitor it using oozie job -log commands, but sometimes it appears that OOZIE-1700 causes oozie to report failure when jobs actually succeed, so for now, we suggest monitoring the actual YARN history server).

# Using Solr #


The simplest way to start with SOLR is using a tarball.  For this example, we will download the CDH5 Tarball and untar it to /opt/solr.

The SOLR tarball can be acquired from the CDH5 tarball [downloads page.](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latestCDH-Version-and-Packaging-Information/cdhvd_cdh_package_tarball.htm)

Extract the tarball, and go to the examples/ directory, where you can launch an embedded core.

Now, cd into /opt/solr, to the examples/directory.

Then, find the solrconfig.xml file.  Edit it to contain the following xml: 

    <directoryFactory name="DirectoryFactory" class="org.apache.solr.core.HdfsDirectoryFactory">
    <str name="solr.hdfs.home">glusterfs:///solr</str>
    <str name="solr.hdfs.confdir">/etc/hadoop/conf</str>
    </directoryFactory>

This tells, when launched, to use gluster as the underlying file store.   You also should make sure that **glusterfs-hadoop jar**, and  the **hadoop-common jar**, are both on the classpath of the solr web server.  To do so - you can update the solrconfig.xml jar directives.  You can also copy the jars in at runtime, to be really safe. 

Once your example core with gluster configuration is setup, launch it with the following properties: 

    java -Dsolr.directoryFactory=HdfsDirectoryFactory -Dsolr.lock.type=hdfs -Dsolr.data.dir=glusterfs:///solr -Dsolr.updatelog=glusterfs:///solr -Dlog4j.configuration=file:/opt/solr-4.4.0-cdh5.0.2/example/etc/logging.properties -jar start.jar 


This starts a basic SOLR server on port 8983.

From here, you will want to make sure you are plugged into the gluster file system for backend storage. 

If the launch works, then you will be able to see a healthy SOLR core running on `http://<SERVER>:8983/solr/`

*Testing SOLR with the embedded examples*

Now that solr is running, you can test it by running the examples which are embedded in the distribution. 

` cd /opt/solr-4.4.0-cdh5.0.2/example/exampledocs && curl http://mrg42:8983/solr/update?commit=true -H "Content-Type: text/xml" -d "@mem.xml" `

