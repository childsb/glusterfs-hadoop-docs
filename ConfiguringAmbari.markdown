#Installing and Configuring Red Hat Storage (RHS)#

In order to configure Hortonworks Data Platform (HDP) 2 to run on Red Hat Storage, you will need to follow the instructions below:

* Identify a set of servers on which you wish to create a Red Hat Storage volume. Note: Each server should comply with requirements for RHS outlined in the RHS Installation and Configuration guide and should have a RAID 6 volume available to be configured as a brick on the RHS server.

* Login to the Red Hat Network and download the RHS ISO from the following location: 
https://rhn.redhat.com/rhn/software/downloads/SupportedISOs.do?filter_string=red%20hat%20storage

* Install the ISO and specify the FQDN for each server but do not configure the brick. Note: Each server must have a FQDN specified, a hostname alone will not meet the requirements for the HDP Ambari deployment tool.

* Register each server with the Red Hat Network, using rhn_register. This is required for the HDP Ambari deployment tool to be able resolve the dependencies of all the packages it will deploy.


##Creating and Configuring your Gluster Volume

* Designate the server in your cluster that will become your Management Server and the server upon which you will install the HDP Ambari Management tool. This is the same server which will deploy your Hadoop stack to all the nodes in your cluster (which is the same thing as the RHS trusted storage pool for your volume).

* On your Management server, open a terminal and change directory to /usr/share/rhs-hadoop-install-$version/   Note: If this does not exist, it is because you have not yum installed rhs-hadoop-install. 

* Edit the hosts.example file and replace the sample hosts with all the hosts within your cluster. Then rename "hosts.example" to "hosts".

* Setup passwordless SSH from your Management Server to every server listed within your hosts file

* Identify the name of your RAID 6 volume on the filesystem for each server, this is usually /dev/sdb

* Create and Configure a Distributed Replicated 2 RHS volume for Hadoop by running:

`./install.sh <NameOfRAID6Volume>`

for example

`./install.sh /dev/sdb`

* Once the installer has finished, verify the volume was created successfully by typing "mount" on each server and ensuring you see /mnt/glusterfs in the list of mounts. In addition, you can type "gluster volume info" to ensure the volume has started and that all the expected nodes in the cluster are present in the volume.

##Installing the Red Hat Storage Hadoop FileSystem Plugin

Download the latest plugin release from http://rhbd.s3.amazonaws.com/maven/index.html and copy it to /usr/lib/hadoop/lib on all the machines within the cluster. Note: When the Red Hat Storage 2.1.1 ISO with the plugin becomes available, the ISO will already have the plugin in this location, so this step will not be necessary.

`mkdir -p /usr/lib/hadoop/lib`
`cd /usr/lib/hadoop/lib`
`wget $url_of_plugin`

-------------------------------

# Configuring Ambari Hadoop 1.x #

These instructions will provide you step-by-step instructions for using Ambari to configure your Hadoop 1.x cluster on top of GlusterFS.

**Note: These instructions will only work on RHEL. Apache Ambari does not yet work on Fedora **

These instructions assume you have already [installed and configured GlusterFS](https://forge.gluster.org/hadoop/pages/Configuration)

##Download the Ambari RPMS

Use the following links to dowload the Ambari RPMs:
[Ambari-server](https://s3-us-west-1.amazonaws.com/rhbd/glusterfs-ambari/ambari-server-1.3.0-SNAPSHOT20130904172038.noarch.rpm)
[Ambari-agent](https://s3-us-west-1.amazonaws.com/rhbd/glusterfs-ambari/ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm) 
to the /tmp directory on your cluster management node.

Then copy the Ambari-agent RPM to all the other nodes in your cluster.

##Pre-installation setup

Once you have copied the rpms perform the following steps on each node:

0. Set permissions for the getfattr process
    vi /etc/sudoers.d/gluster
    mapred ALL= NOPASSWD: /usr/bin/getfattr
    chmod 440 /etc/sudoers.d/gluster

1. Navigate to the proper directory
    cd /tmp

2. Install the EPEL repo
    yum install epel-release

3. Install the ambari-server and ambari-agent rpms
    yum install ambari-server-1.3.0-SNAPSHOT20130904172038.noarch.rpm
    yum install ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm

4. On the server node run the setup program:
    ambari-server setup -s _(this will install the server with the current defaults)_

5. On the agent nodes (including the server node) configure the Ambari Agent by editing the ambari-agent.ini file:
   vi  /etc/ambari-agent/conf/ambari-agent.ini
          [server]
          hostname={your.ambari.server.hostname}
          url_port=8440
          secured_url_port=8441

6. Start the Ambari Server
    ambari-server start

7. Start the Ambari Agent
    ambari-agent start

##Ambari wizard setup
Once you have started the Ambari-server and Ambari agent open a browser to:
http://your.ambari.server.hostname:8080
Ambari Hadoop
Once the login screen loads use admin/admin for the credentials and press the Sign In button.

1. Welcome: Assign a name to your Cluster
2. Select Stack: Select the HDP 1.3.2-GlusterFS Stack
3. Install Options: 
	--Target Hosts: Enter one host per line including the hosts used for the install if it is part of the Storage Pool. 
	--Host Registration Information: Select ‘Perform 	Manual Registration on hosts, do not use SSH’.
	(You will get a pop of warning. The manual install of the agents was completed by the install.sh script.  No additional action needs to take place.  Select OK and continue)

	--Advanced Options: Do not select any values in this section. 
          Select Next to continue to the following screen.  
          (You will receive another pop-up about manually registering 
          your Ambari-agents.  This has already been completed, select OK
          to continue.)

4. Confirm Hosts: Confirm all the hosts you entered from the previous page are present. The system will now register the hosts in the cluster.  Select Next once the registration process is complete.

5. Choose Services: First press the minimum link to remove any unnecessary services. Then select GlusterFS and MapReduce.  Then click next.  Note, if you do not select Nagios and/or Ganglia you will receive a pop-up warning of limited functionality.  Select OK to continue.

6. Assign Masters: Chose which services you want to run on each host.  Select Next when finished

7. Assign Slaves and Clients: Chose which components you want on each hosts. Select the check box to put the GlusterFS client on all the nodes.

8. Customize Services: For the GlusterFS tab you can accept the defaults. If you have chosen to install Nagios you will need to enter a password and email for alerts.  Under the MapReduce tab set the value for mapred.local.dir to /mnt/brick1/mapredlocal.  Select Next to continue.

9. Review: Make sure the services are what you selected in the previous steps. Select Deploy to continue.  You are done once the deploy process completes!

--------------------------------

# Configuring Ambari Hadoop 2.x #

* On every server within the cluster, add the Ambari repo by running the following command:
`rpm -Uvh http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.1.4.4-1.x/repos/centos6/AMBARI.1.4.4-1.x-1.el6.noarch.rpm`

* Verify the Ambari Repo was successfully added, by running:

`yum repolist` 

   You should see the Ambari packages in the list.

* Install the Ambari Server and Agent on the Management Server:

Note: The Ambari Server and Agents must be manually installed as the Ambari Repo Alpha does not contain a working Ambari Server and Agent.

On the server intended to be your Ambari Server, please yum install the [Ambari Server](http://ambari-fork.s3.amazonaws.com/ambari-server-1.3.0-SNAPSHOT20140110162116.noarch.rpm) and the [Ambari Agent](http://ambari-fork.s3.amazonaws.com/ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm) from our S3 Repo.

Navigate to the directory where you downloaded the RPMs and run:
`yum install ambari-server-1.3.0-SNAPSHOT20140110162116.noarch.rpm`
`yum install ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm`

* On the management server, activate the GlusterFS enabled HDP Stack in Ambari by editing the /var/lib/ambari-server/resources/stacks/HDP/2.0.6.GlusterFS/metainfo.xml and setting the active flag to true:
`     <metainfo>`
`        <versions>`
`               <active>true</active>`
`         </versions>`
`     </metainfo>`

* On the **management server **, start the Ambari Server:
`     ambari-server setup -s`
`     ambari-server start`

*  Install  **only the Ambari Agent **  on  **all servers ** within your cluster:
`     yum install ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm`

* Configure the Agent with the Ambari Server Hostname: 

`     sed -i 's/'localhost'/<MgmtNode>/' /etc/ambari-agent/conf/ambari-agent.ini`

(where <MgmtNode> is your managment host name ie. hwx17.rhs)

Now, On **EACH node which will serve as a hadoop slave** Start the Ambari Agent: 
 
`    ambari-agent start`




## Deploying and Configuring the HDP Stack on Red Hat Storage


* Launch a browser and enter the following in the URL by replacing `<hostname>` with the hostname of your ambari server 

`http://<hostname>:8080`

* Enter "admin" and "admin" for the login and password.

* Assign your cluster a name, such as "MyCluster"

* Select the "HDP 2.0.6.GlusterFS" stack

* Enter your target hosts using their FQDN and select the "Perform Manual Registration  on hosts and not use SSH" radio button. Click on the the "Register and Confirm" button and Ignore the Warnings.

* Select the services within the stack that you would like to install (YARN, Nagios, etc.)

* On the "Assign Masters" screen, specify the same server for all the services. This is often the Ambari Management server.

* On the "Assign Slaves and Clients" screen, click "all" for both the NodeManagers and the Clients.

* On the "Customize Services" screen, click on the YARN tab, expand the "Advanced" section and set the "yarn.log.server.url" to "http://localhost:19888/jobhistory/nmlogs" 

* On the "Customize Services" screen, click on the YARN tab, expand the "Node Manager" section and remove any entries that refer to /mnt/glusterfs in the "yarn.nodemanager.log-dirs" and "yarn.nodemanager.local-dirs" properties.

* On the "Customize Services" screen, click on the MapReduce2 tab, scroll down to the bottom and under the "custom mapred-site.xml" add the following 4 custom properties and then click on the "Next" button.

`mapred.healthChecker.script.path=glusterfs:///mapred/jobstatus`
`mapred.job.tracker.history.completed.location=glusterfs:///mapred/history/done`
`mapred.system.dir=glusterfs:///mapred/system`
`mapreduce.jobtracker.staging.root.dir=glusterfs:///user`

* Review your configuration and then click the "Deploy" button. The services should deploy successfully with a few warnings and take you through to the Ambari Dashboard. You should see all the services started with the exception of Nagios. Note: If you select the YARN service, you will notice that the NodeManagers are not yet running. This is because the Hadoop Linux Container Executor for the NodeManagers still needs to be configured.



## Configuring the Linux Container Executor (LCE)



Note that amabri sets up linux containers for you, but for hadoop 2.3.0,  you need to ensure that it is running:

` org.apache.hadoop.yarn.server.nodemanager.GlusterContainerExecutor 

Or that you have enabled Kerberos security in order to operate the glusterfs-hadoop plugin with normal multitenancy.



* In the Ambari Dashboard, select the YARN service and then click the "Stop-All" button.

* On the management server, open a terminal and change directory to /usr/share/rhs-hadoop-install-$version/   Note: If this does not exist, it is because you have not yum installed rhs-hadoop-install. Copy the setup_container_executor.sh script to all servers within the cluster and run it:
`./setup_container_executor.sh`

* For each server within the cluster, edit the /etc/hadoop/conf/container-executor.cfg file and replace the contents, with the following:
`yarn.nodemanager.linux-container-executor.group=hadoop`
`banned.users=yarn`
`min.user.id=1000`
`allowed.system.users=tom`

Note: Make sure there is no additional whitespace at the end of each line or at the end of the file. Also, "tom" is an example user. You need to explicitly add each user, in a comma delimited list, that is allowed to use the cluster, as the value for the allowed.system.users parameter. For each user that add, that same user needs to exist on every node on the cluster. This can be relatively easily if you are using LDAP for authentication, but if you are not, you will need to run the following command on each server in the cluster, for each user:
`useradd -g hadoop tom`

* In the Ambari Dashboard, select the YARN service and then click the "Start-All" button. Note: Both stopping and starting the services can take some time.

## Using Hadoop 

* To test your cluster, open a terminal window and navigate to /usr/lib/hadoop. Then su to one of the configured allowed.system.users (such as tom) and submit a Hadoop Job:

`bin/hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples-2.2.0.2.0.6.0-101.jar teragen 1000 in`


##Common Errors

* If you see an exception stating that "job.jar changed on src filesystem" it means that you need to synchronize the clocks across your cluster. You can do this by running the following command on each node:
`ntpd -qg`

------------------

#Multiple Users in Ambari#

* Stop the hadoop services

* Run this script on the node:

`#!/bin/sh`

`          gluster_mount=/mnt/glusterfs`
`          process_user=yarn`
`          process_group=hadoop`
`          yarn_nodemanager_remote_app_log_dir="/tmp/logs"`
`          mapreduce_jobhistory_intermediate_done_dir="/mr_history/tmp"`
`          mapreduce_jobhistory_done_dir="/mr_history/done"`
`          mapreduce_jobhistory_apps_logs="/app-logs"`
`          yarn_staging_dir=/job-staging-yarn/`
`          task_controler=/usr/lib/hadoop-yarn/bin/container-executor`
`          task_cfg=/etc/hadoop/conf/container-executor.cfg`


`          setPerms(){`
`            Paths=("${!1}")`
`            Perms=("${!2}")`
`            Root_Path=$3`
`            User=$4`
`            Group=$5`

`            for (( i=0 ; i<${#Paths[@]} ; i++ ))`
`             do`
`              mkdir -p ${Root_Path}/${Paths[$i]}`
`              chown ${User}:${Group}  ${Root_Path}/${Paths[$i]}` 
`              chmod ${Perms[$i]} ${Root_Path}/${Paths[$i]}` 
`              echo ${Paths[$i]} ${Perms[$i]}`
`             done` 
`          }`

`          echo "Setting permissions on Gluster Volume located at ${gluster_mount}"`

`          paths=("/tmp" "/user" "/mr_history" "${yarn_nodemanager_remote_app_log_dir}"    "${mapreduce_jobhistory_intermediate_done_dir}" "${mapreduce_jobhistory_done_dir}" "/mapred" "${yarn_staging_dir}" "${mapreduce_jobhistory_apps_logs}");`
`          perms=(1777 0775 0755 1777 1777 0750 0770 2770 1777);`
`          setPerms paths[@] perms[@] ${gluster_mount} ${process_user} ${process_group}`

`          echo "Setuid bit on task controller"`
`          chown root:${process_group} ${task_controler} ; chmod 6050 ${task_controler}`
`          chown root:${process_group} ${task_cfg}`

* modify my /etc/hadoop/conf/container-executor to look like:

`          yarn.nodemanager.linux-container-executor.group=hadoop `
`          banned.users=yarn min.user.id=1000 `
`          allowed.system.users=tom`

-----------------------

#Configuring Hadoop 2.x without Ambari#

##Download and Extract Hadoop

Navigate to [Hadoop Download Page](http://hadoop.apache.org/releases.html#Download) and download the hadoop-2.1.0-beta.tar.gz  tar-ball (the most recent 2.x release at the time this document was created) to the /opt directory on the Master server. Extract the tar ball in the /opt directory.

For the sake of this document $HADOOP_HOME is the directory the tarball extracts to under /opt/

**Configure the Plugin**

Copy the plugin jar to $HADOOP_HOME/share/hadoop/common/lib/. [The plugin jar can be obtained here.](http://rhbd.s3.amazonaws.com/maven/indexV2.html)

##Modify the hadoop-env.sh file

Edit the $HADOOP_HOME/etc/hadoop/hadoop-env.sh file to reflect JAVA_HOME. Remove the # preceding the line and set the JAVA_HOME to the correct directory. This would usually be:

 export JAVA_HOME=/usr/lib/jvm/java-1.6.0/

** Modify the mapred-site.xml **

Navigate to $HADOOP_HOME/etc/hadoop and modify the mapred-site.xml to reflect the following:

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`
`<property>`
`<name>yarn.nodemanager.linux-container-executor.group</name>`
`<value>hadoop</value>`
`</property>`

`<property>`
`<name>yarn.nodemanager.container-executor.class</name> `
`<value>org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor</value>`
`</property>`
` <configuration>`
`   <property>`
`     <name>mapreduce.framework.name</name>`
`     <value>yarn</value>`
`   </property>`

`  <property>`
`    <name>yarn.app.mapreduce.am.staging-dir</name>`
`    <value>glusterfs:///user</value>`
`  </property>`

`  <property>`
`    <name>mapred.healthChecker.script.path</name>`
`    <value>glusterfs:///mapred/jobstatus</value>`
`  </property>`

`  <property>`
`    <name>mapred.job.tracker.history.completed.location</name>`
`    <value>glusterfs:///mapred/history/done</value>`
`  </property>`

`  <property>`
`    <name>mapred.system.dir</name>`
`    <value>glusterfs:///mapred/system</value>`
`  </property>`

`  <property>`
`    <name>mapreduce.jobtracker.staging.root.dir</name>`
`    <value>glusterfs:///user</value>`
`  </property>`

`</configuration>`

## Modify the core-site.xml

Navigate to $HADOOP_HOME/etc/hadoop) and modify the core-site.xml to reflect the following (note: node-1 needs to be replaced a server in your storage pool):

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

`<configuration>`
` <property>`
`  <name>fs.defaultFS</name>`
`  <value>glusterfs:///</value>`
` </property>`

` <property>`
`  <name>fs.default.name</name>`
`  <value>glusterfs:///</value>`
` </property>`

` <property>`
`  <name>fs.AbstractFileSystem.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.local.GlusterFs</value>`
` </property>`

` <property>`
`  <name>fs.glusterfs.impl</name>`
`  <value>org.apache.hadoop.fs.glusterfs.GlusterFileSystem</value>`
` </property>`

`<!-- two volume setup. gv0 and gv1 -->`
`<property>`
`<name>fs.glusterfs.volumes</name>`
`<value>gv0,gv1</value>`
`</property>`

`<property>`
`<name>fs.glusterfs.volume.fuse.gv0</name>`
`<value>/mnt/gv0</value>`
`</property>`

`<property>`
`<name>fs.glusterfs.volume.fuse.gv1</name>`
`<value>/mnt/gv1</value>`
`</property>`


`</configuration>`


## Modify the yarn-site.xml

Navigate to $HADOOP_HOME/etc/hadoop) and modify the yarn-site.xml to reflect the following (note: node-1 needs to be replaced with the server designated as your ResourceManager (Master) in your storage pool):

`<?xml version="1.0"?>`
`<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>`

`<!-- Put site-specific property overrides in this file. -->`

`<configuration>`
`<property>`
`    <name>yarn.nodemanager.aux-services</name>`
`    <value>mapreduce_shuffle</value>`
`    <description>Auxilliary services of NodeManager</description>`
`  </property>`

`  <property>`
`    <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>`
`    <value>org.apache.hadoop.mapred.ShuffleHandler</value>`
 ` </property>`

`  <property>`
`    <name>yarn.resourcemanager.resource-tracker.address</name>`
`    <value>node-1:8025</value>`
`  </property>`

`  <property>`
`    <name>yarn.resourcemanager.scheduler.address</name>`
`    <value>node-1:8030</value>`
`  </property>`

`  <property>`
`    <name>yarn.resourcemanager.address</name>`
`    <value>node-1:9040</value>`
`  </property>`
` </configuration>`

## Configure for Multiple Users

Setup container executor by creating the following script (in $HADOOP_HOME):

---------------------------

    #!/bin/sh
    HADOOP_HOME=/opt/hadoop
    process_user=yarn
    process_group=hadoop
    task_controller=$HADOOP_HOME/bin/container-executor
    task_cfg=$HADOOP_HOME/etc/hadoop/container-executor.cfg

    echo "Configuring the Linux Container Executor for Hadoop"
    chown root:${process_group} ${task_controller} 
    chmod 6050 ${task_controller}
    chown root:${process_group} ${task_cfg}
    chown -R ${process_user}:${process_group} ${HADOOP}/logs

------------------------------

Set the $HADOOP_HOME/etc/hadop/container-executor.cfg to:
    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom

** Synchronize the configuration across the cluster **

SCP the $HADOOP_HOME directory to each server in the  the cluster, for example:
    scp -r $HADOOP_HOME/ root@svr2:/opt/
    scp -r $HADOOP_HOME/ root@svr3:/opt/
    scp -r $HADOOP_HOME/ root@svr4:/opt/
    etc...
 
Then ssh to each node, and run the container-executor script.



##Starting Hadoop

On the Master server, open a terminal window, navigate to $HADOOP_HOME and run the following commands:
    sbin/yarn-daemon.sh start resourcemanager
    sbin/yarn-daemon.sh start nodemanager
    sbin/mr-jobhistory-daemon.sh start historyserver

On each Slave server, open a terminal window, navigate to $HADOOP_HOME and run the following commands:

    sbin/yarn-daemon.sh start nodemanager

##Verifying Hadoop is running successfully on GlusterFS

In a browser, launch the YARN ResourceManager UI by navigating to http://<$MasterHostName>:8088/cluster/nodes

Verify that all the hosts listed within your storage pool have reported in. If a host is missing, shell into that host and take a look at the files within $HADOOP_HOME/logs to check for errors when the process attempted to start.

** Running a sample Hadoop Job **

Navigate to $HADOOP_HOME and run the following:

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar teragen 10000 in-dir

and then once TeraGen has complete, run

bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar terasort in-dir out-dir

