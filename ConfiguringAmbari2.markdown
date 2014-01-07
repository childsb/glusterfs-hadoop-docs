In order to configure Hortonworks Data Platform (HDP) 2 to run on Red Hat Storage, you will need to follow the instructions below:

** Installing and Configuring Red Hat Storage (RHS) **

* Identify a set of servers on which you wish to create a Red Hat Storage volume. Note: Each server should comply with requirements for RHS outlined in the RHS Installation and Configuration guide and should have a RAID 6 volume available to be configured as a brick on the RHS server.

* Login to the Red Hat Network and download the RHS ISO from the following location: 
https://rhn.redhat.com/rhn/software/downloads/SupportedISOs.do?filter_string=red%20hat%20storage

* Install the ISO and specify the FQDN for each server but do not configure the brick. Note: Each server must have a FQDN specified, a hostname alone will not meet the requirements for the HDP Ambari deployment tool.

* Register each server with the Red Hat Network, using rhn_register. This is required for the HDP Ambari deployment tool to be able resolve the dependencies of all the packages it will deploy.

** Creating and Configuring your Gluster Volume **

* Designate the server in your cluster that will become your Management Server and the server upon which you will install the HDP Ambari Management tool. This is the same server which will deploy your Hadoop stack to all the nodes in your cluster (which is the same thing as the RHS trusted storage pool for your volume).

* On your Management server, open a terminal and change directory to /usr/share/rhs-hadoop-install-0_50/

* Edit the hosts.example file and replace the sample hosts with all the hosts within your cluster. Then rename "hosts.example" to "hosts".

* Setup passwordless SSH from your Management Server to every server listed within your hosts file

* Identify the name of your RAID 6 volume on the filesystem for each server, this is usually /dev/sdb

* Create and Configure a Distributed Replicated 2 RHS volume for Hadoop by running:

`./install.sh <NameOfRAID6Volume>`

for example

`./install.sh /dev/sdb`

* Once the installer has finished, verify the volume was created successfully by typing "mount" on each server and ensuring you see /mnt/glusterfs in the list of mounts. In addition, you can type "gluster volume info" to ensure the volume has started and that all the expected nodes in the cluster are present in the volume.

** Installing and Configuring HDP **


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

* Install the Ambari Server on your Management Server :    

`yum -y install ambari`

* On the management server, activate the GlusterFS enabled HDP Stack in Ambari:

   Edit /var/lib/ambari-server/resources/stacks/HDP/2.0.6.GlusterFS/metainfo.xml

   Set the active flag to true:
     
`     <metainfo>`
`        <versions>`
`               <active>true</active>`
`         </versions>`
`     </metainfo>`

* On the management server, start the Ambari Server:

`     ambari-server setup -s`
`     ambari-server start`


* Follow these instructions on all other servers within your cluster:

 Install the Ambari Agent :    


     yum -y install ambari-agent

1. Configure the Agent with the Ambari Server Hostname: 


     sed -i 's/'localhost'/<managementnodename>/' /etc/ambari-agent/conf/ambari-agent.ini
     (where <managementnodename> is your managment host name ie. hwx17.rhs)


2. Start the Ambari Agent 
 
     ambari-agent start



Using Apache Ambari to deploy your HDP Stack on Red Hat Storage


3. Launch a browser and enter the following in the URL by replacing <hostname> with the hostname of your ambari server 


http://<hostname>:8080


4. Enter "admin" and "admin" for the login and password.

5. Assign your cluster a name

6. select the "HDP 2.0.6.GlusterFS" stack
(Follow the standard Ambari instuctions paste the registration point)

7. select the services you woudl like to install

8. Assign these values to each service:

# Yarn Config # 

yarn.log.server.url http://localhost:19888/jobhistory/nmlogs

# MapReduce Config # 

Change map reduce settings:

mapreduce.jobhistory.done-dir = glusterfs://

<name>mapreduce.jobhistory.intermediate-done-dir</name>
<value>glusterfs:///mr_history/tmp</value>


IN the advanced section:
 <property>
<name>yarn.app.mapreduce.am.staging-dir</name>
<value>glusterfs:///tmp/hadoop-yarn/staging/mapred/.staging</value>
</property>

mapreduce.jobhistory.intermediate-done-dir = glusterfs:///mr-history/tmp
mapreduce.jobhistory.done-dir = glusterfs:///mr-history/done

Add these as map-reduce custom properties:
<property>
<name>mapred.healthChecker.script.path</name>
<value>glusterfs:///mapred/jobstatus</value>
</property>

<property>
<name>mapred.job.tracker.history.completed.location</name>
<value>glusterfs:///mapred/history/done</value>
</property>

<property>
<name>mapred.system.dir</name>
<value>glusterfs:///mapred/system</value>
</property>

<property>
<name>mapreduce.jobtracker.staging.root.dir</name>
<value>glusterfs:///user</value>
</property>

</configuration>