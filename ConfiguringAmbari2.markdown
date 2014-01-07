Here's how to configure Ambari with GlusterFS and Hadoop 2.x.

#Pre-Setup#


1. Add the Ambari repo by creating the following file: /etc/yum.repos.d/ambari.repo and adding the following contents to it:


[AMBARI.bw-m15-1.x]
name=Ambari 1.x
baseurl=http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.bw-m15-1.x/repos/centos6
gpgcheck=1
gpgkey=http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.bw-m15-1.x/repos/centos6/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


[HDP-UTILS-1.1.0.16]
name=Hortonworks Data Platform Utils Version - HDP-UTILS-1.1.0.16
baseurl=http://s3.amazonaws.com/dev.hortonworks.com/HDP-UTILS-1.1.0.16/repos/centos6
gpgcheck=0
gpgkey=http://s3.amazonaws.com/dev.hortonworks.com/HDP-UTILS-1.1.0.16/repos/centos6/RPM-GPG-KEY/RPM-GPG-KEY-Jenkins
enabled=1
priority=1


2. Verify the Ambari Repo was successfully added, by running:


   yum repolist 


   You should see the Ambari packages in the list.


# Setting up the Ambari Server #


1. Install the Ambari Server on your Management Server :    


   yum -y install ambari*


Note: The Ambari packages have dependencies, such as postgresql-server. At present, you will need to rhn_register your server in order for it to have access to the Red Hat Network so that these dependencies can be resolved and installed. 


2. On the management server, activate the GlusterFS enabled HDP Stack in Ambari:


   Edit /var/lib/ambari-server/resources/stacks/HDP/2.0.6.GlusterFS/metainfo.xml


   Set the active flag to true:
     
     <metainfo>
         <versions>
               <active>true</active>
         </versions>
     </metainfo>



3. On the management server, start the Ambari Server:


     ambari-server setup -s
     ambari-server start



# Setting up the Ambari Agents #

Follow these instructions on all other servers within your cluster:

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