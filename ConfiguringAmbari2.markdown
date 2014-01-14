In order to configure Hortonworks Data Platform (HDP) 2 to run on Red Hat Storage, you will need to follow the instructions below:

** Installing and Configuring Red Hat Storage (RHS) **

* Identify a set of servers on which you wish to create a Red Hat Storage volume. Note: Each server should comply with requirements for RHS outlined in the RHS Installation and Configuration guide and should have a RAID 6 volume available to be configured as a brick on the RHS server.

* Login to the Red Hat Network and download the RHS ISO from the following location: 
https://rhn.redhat.com/rhn/software/downloads/SupportedISOs.do?filter_string=red%20hat%20storage

* Install the ISO and specify the FQDN for each server but do not configure the brick. Note: Each server must have a FQDN specified, a hostname alone will not meet the requirements for the HDP Ambari deployment tool.

* Register each server with the Red Hat Network, using rhn_register. This is required for the HDP Ambari deployment tool to be able resolve the dependencies of all the packages it will deploy.

** Creating and Configuring your Gluster Volume **

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

* [Download](http://rhbd.s3.amazonaws.com/steve/post_install_dirs.sh) and run the post_install_dirs.sh script on each server in the cluster.

** Installing the Red Hat Storage Hadoop FileSystem Plugin **

Download the latest plugin release from http://rhbd.s3.amazonaws.com/maven/index.html and copy it to /usr/lib/hadoop/lib on all the machines within the cluster. Note: When the Red Hat Storage 2.1.1 ISO with the plugin becomes available, the ISO will already have the plugin in this location already, so this step will not be necessary.

** Installing and Configuring Apache Ambari **

* Add the Ambari repo by creating the following file: /etc/yum.repos.d/ambari.repo and adding the following contents to it:
		
`[AMBARI.bw-m15-1.x]`
`name=Ambari 1.x`
`baseurl=http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.bw-m15-1.x/repos/centos6`
`gpgcheck=1`
`gpgkey=http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.bw-m15-1.x/repos/centos6/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins`
`enabled=1`
`priority=1`
		
`[HDP-UTILS-1.1.0.16]`
`name=Hortonworks Data Platform Utils Version - HDP-UTILS-1.1.0.16`
`baseurl=http://s3.amazonaws.com/dev.hortonworks.com/HDP-UTILS-1.1.0.16/repos/centos6`
`gpgcheck=0`
`gpgkey=http://s3.amazonaws.com/dev.hortonworks.com/HDP-UTILS-1.1.0.16/repos/centos6/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins`
`enabled=1`
`priority=1`

* Verify the Ambari Repo was successfully added, by running:

`yum repolist` 

   You should see the Ambari packages in the list.

* Install the Ambari Server and Agent on your Management Server :    

Note: The working glusterfs stack will only be shipped by Hortonworks with HDP 1.4.3.1, this is not internally or externally available yet. In the meantime, please yum install the [http://ambari-fork.s3.amazonaws.com/ambari-server-1.3.0-SNAPSHOT20140110162116.noarch.rpm](Ambari Server) and [http://ambari-fork.s3.amazonaws.com/ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm](Agent) from our S3 Repo:

* On the management server, activate the GlusterFS enabled HDP Stack in Ambari by editing the /var/lib/ambari-server/resources/stacks/HDP/2.0.6.GlusterFS/metainfo.xml and setting the active flag to true:
`     <metainfo>`
`        <versions>`
`               <active>true</active>`
`         </versions>`
`     </metainfo>`

* On the management server, start the Ambari Server:
`     ambari-server setup -s`
`     ambari-server start`

*  Install only the Ambari Agent on all other servers within your cluster:
`     yum -y install ambari-agent`

* Configure the Agent with the Ambari Server Hostname: 

`     sed -i 's/'localhost'/<managementnodename>/' /etc/ambari-agent/conf/ambari-agent.ini`

(where <managementnodename> is your managment host name ie. hwx17.rhs)

Start the Ambari Agent 
 
`    ambari-agent start`


** Deploying and Configuring the HDP Stack on Red Hat Storage **


* Launch a browser and enter the following in the URL by replacing `<hostname>` with the hostname of your ambari server 

`http://<hostname>:8080`

* Enter "admin" and "admin" for the login and password.

* Assign your cluster a name, such as "MyCluster"

* Select the "HDP 2.0.6.GlusterFS" stack

* Select the services within the stack that you would like to install and accept the defaults for the configuration parameters, with the exception of the MapReduce Tab. For MapReduce, scroll down to the bottom and add the following 4 custom properties:

`mapred.healthChecker.script.path=glusterfs:///mapred/jobstatus`
`mapred.job.tracker.history.completed.location=glusterfs:///mapred/history/done`
`mapred.system.dir=glusterfs:///mapred/system`
`mapreduce.jobtracker.staging.root.dir=glusterfs:///user`

* The services should deploy successfully with a few warnings and take you through to the Ambari Dashboard. You should see all the services started with the exception of Nagios and MapReduce2.

** Starting the MapReduce2 service **

* [Download](http://rhbd.s3.amazonaws.com/steve/setup_container_executor.sh) and run the setup_container_executor.sh script on each server in the cluster.

* For each server within the cluster, edit the /etc/hadoop/conf/container-executor.cfg file and replace the contents, with the following:
`yarn.nodemanager.linux-container-executor.group=hadoop`
`banned.users=yarn`
`min.user.id=1000`
`allowed.system.users=tom`

Note: Make sure there is no additional whitespace at the end of each line or at the end of the file. Also, "tom" is an example user. You need to explicitly add each user, in a comma delimited list, that is allowed to use the cluster, as the value for the allowed.system.users parameter. For each user that add, that same user needs to exist on every node on the cluster. This can be relatively easily if you are using LDAP for authentication, but if you are not, you will need to run the following command on each server in the cluster, for each user:
`useradd -g hadoop tom`

* Go back to the Ambari Web UI Dashboard. Select the YARN Service and click the "stop-all" button. Once all the services have stopped, Select the "start-all" button. Both stopping and starting the services can take some time.

** Using Hadoop  **

* To test your cluster, open a terminal window and navigate to /usr/lib/hadoop. Then su to one of the configured allowed.system.users (such as tom) and submit a Hadoop Job:

`bin/hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples-2.2.0.2.0.6.0-101.jar teragen 1000 in`


** Common Errors  **

* If you see an exception stating that "job.jar changed on src filesystem" it means that you need to synchronize the clocks across your cluster. You can do this by running the following command on each node:
`ntpd -qg`
------------------