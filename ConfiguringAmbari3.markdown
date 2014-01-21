Fast directions

** Before you start ** 

* **Identify a set of servers** on which you wish to create a Red Hat Storage volume. 
* Note: Each server should comply with **requirements for RHS **outlined in the RHS Installation and Configuration guide and should have a **RAID 6**volume available to be configured as a brick on the RHS server.
* Login to the Red Hat Network  and **download the RHS ISO **from the following location: 
https://rhn.redhat.com/rhn/software/downloads/SupportedISOs.do?filter_string=red%20hat%20storage

---------------------------------------

FOR EACH SERVER Install the ISO (see below):

* **Specify the FQDN **for each server but...
*  **DO NOT**configure the brick. 
* **EACH server must have a FQDN specified (i.e. `hadoop1.lab.test`, NOT `hadoop1`)**, otherwise, Hadoop services wont work properly.
* Register with the Red Hat Network, using rhn_register. 
    * **REGISTRATION IS REQUIRED**for the HDP Ambari deployment (otherwise, all packages wont be resolved)

---------------------------------------

** SETUP GLUSTER VOLUME **

* Designate **MgmtSrvr**:
   * This server: 
       * Will setup gluster brick / RHS Trusted Storage pool to your nodes.
       * Will also install the HDP Ambari Management tool. 
* On your **MgmtSrvr**: 
    * open a terminal 
    * change directory to /usr/share/rhs-hadoop-install-$version/
        * If this DOES NOT exist, it is because YOU DID NOT "yum install rhs-hadoop-install".

* On your **MgmtSrvr**: 
    * Open and **EDIT the hosts.example file**as documented. 
    * **RENAME**it to  "hosts.example" to "hosts".  

This file will get copied to /etc/hosts, must match ambari hosts defined later on exactly.  

* Setup **PASSWORDLESS SSH**from your Management Server to every server listed within your hosts file.
    * Easy way to do this : ( cat your master node id_rsa.pub to "authorized keys" in slaves )

* Identify the **name of your RAW DISK (i.e. RAID 6 volume)**on the filesystem for each server, this is usually /dev/sdb . If using 

---------------------------------------

* Create and Configure a **Distributed Replicated 2 NODE RHS **volume for Hadoop by running:
   ./install.sh NameOfRAID6Volume, for example:
                 ./install.sh /dev/sdb # 

* Wait for installer to finish.
    * verify the volume was created successfully. 
        *   **Type `mount`**on each server, ensure that you see /mnt/glusterfs in the list of mounts. 
        *   **Type `gluster volume info`**, ensure that you see "HadoopVol" (the volume name), with Bricks online:
---------------------------------------

**Installing the Red Hat Storage Hadoop FileSystem Plugin**

* Download the latest plugin release from http://rhbd.s3.amazonaws.com/maven/index.html 
* copy it to /usr/lib/hadoop/lib on all the machines within the cluster. 
    * Note: When the Red Hat Storage 2.1.1 ISO with the plugin becomes available, the ISO will already have the plugin in this location already, so this step will not be necessary.

`mkdir -p /usr/lib/hadoop/lib ; cd /usr/lib/hadoop/lib ; wget $url_of_plugin`

---------------------------------------

**Installing and Configuring Apache Ambari**

* FOR EACH SERVER
    * add the Ambari repo by running the following command:        
        `rpm -Uvh http://s3.amazonaws.com/dev.hortonworks.com/AMBARI.1.4.4-1.x/repos/centos6/AMBARI.1.4.4-1.x-1.el6.noarch.rpm`
    * Verify the Ambari Repo was successfully added, by running: `yum repolist` (you now should see AMBARI-**packages).

    * Install the Ambari Server and Agent on the Management Server:
        * Note: The Ambari Server and Agents must be manually installed as the Ambari REPO ALPHA does not contain a working Ambari Server and Agent.
    
    `wget http://ambari-fork.s3.amazonaws.com/ambari-server-1.3.0-SNAPSHOT20140110162116.noarch.rpm`
    
    `wget http://ambari-fork.s3.amazonaws.com/ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm`
    
    `yum install ambari-server-1.3.0-SNAPSHOT20140110162116.noarch.rpm`
    
    `yum install ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm`

---------------------------------------

**Start the MASTER AMBARI SERVER **

* Activate the GlusterFS enabled HDP Stack ~

* edit : `/var/lib/ambari-server/resources/stacks/HDP/2.0.6.GlusterFS/metainfo.xml`, set "active" to true:
               <active>true</active>
* **Now start the server ! **

 `     ambari-server setup -s`
 `     ambari-server start`

**Start all the AMBARI AGENTS (Hadoop Nodes) in your cluster: **
*  `     yum install ambari-agent-1.3.0-SNAPSHOT20140110162153.x86_64.rpm`
* Configure the Agent with the Ambari Server Hostname: 
    * ` sed -i 's/'localhost'/$MASTER_SERVER/' /etc/ambari-agent/conf/ambari-agent.ini` 
    * `ambari-agent start`
---------------------------------------

**NOW, Configure and DEPLOY your HADOOP CLUSTER ! **

* Launch a browser 

  * enter the following in the URL by replacing `<hostname>` with the hostname of your ambari server `http://<hostname>:8080`

First Page: 

* Enter **"admin" **and **"admin" **for the login and password.

* Assign your cluster a name, such as **"MyCluster" **

* SELECT **"HDP 2.0.6.GlusterFS" **stack

Next page: 

* Enter your TARGET HOSTS using their **FQDN **

* select the **"Perform Manual Registration on hosts and not use SSH"** radio button. 

* Click the **"Register and Confirm" **button ( ignore warnings ).

* Select the **services **within the stack that you would like to install (YARN, Nagios, etc.)

* On the **"Assign Masters" **screen
    * specify the **SAME SERVER **for all the services
    * Typically, pick your **Ambari Management server**for these.

* On the **"Assign Slaves and Clients" **screen, click **"ALL" **for:both 
  * the NodeManagers and 
  * the Clients.

* On the **"Customize Services" **screen, click on the **YARN **tab
    * expand the **"Advanced" **section and set
        * **"yarn.log.server.url" **= "http://localhost:19888/jobhistory/nmlogs" 
    * expand the **"Node Manager" **section 
        * Remove any entries* that refer to **/mnt/glusterfs **in the
            * **"yarn.nodemanager.log -dirs" **OR **"yarn.nodemanager.local-dirs" **properties. 
* Click on the **MapReduce2 **tab, 
    * scroll down to the bottom and 
    * under the **"custom mapred-site.xml" **add the following 4 custom properties 

`mapred.healthChecker.script.path = glusterfs:///mapred/jobstatus`

`mapred.job.tracker.history.completed.location = glusterfs:///mapred/history/done`

`mapred.system.dir =  glusterfs:///mapred/system`

`mapreduce.jobtracker.staging.root.dir= glusterfs:///user`

* Review your configuration 
* click the "Deploy" button. 

Now... what next?  The services should deploy successfully with a **few warnings **and take you through to the Ambari Dashboard. 

You should see all the services started with the exception of Nagios. 
---------------------------------------

**YOUR CLUSTER ISNT READY JUST YET... WAIT !!! **

The NodeManagers are **NOT YET **running. This is because the Hadoop **Linux Container Executor**for the NodeManagers still needs to be configured.  

**THIS IS THE FINAL STEP **!!!

** Configuring the Linux Container Executor (LCE) **

* (In the **DASHBOARD **) Select the **YARN service **and then click the **"Stop-All" **button.

* On the **MANAGEMENT SERVER **, open a terminal 
    * change directory to /usr/share/rhs-hadoop-install-$version/   
    * Note: If this does not exist, it is because you have not yum installed rhs-hadoop-install. 
* ON **EACH NODE **OF YOUR CLUSTER
    * Copy the setup_container_executor.sh script to all servers ** within the cluster and run it:
`./setup_container_executor.sh`

* For **EACH NODE **edit the /etc/hadoop/conf/container-executor.cfg file and replace the contents, with the following:

   `yarn.nodemanager.linux-container-executor.group=hadoop`

   `banned.users=yarn`

   `min.user.id=1000`

   `allowed.system.users=tom,sally`

* Make sure there is  **no additional whitespace **at the end of each line OR **at the end of the file 
* Also, "tom" is an example user. You need to **explicitly add each valida cluster user **to the value of the **allowed.system.users parameter **. 
* For each user that add, that **same user needs to exist **on every node on the cluster, remember that since they are storing data in a  **GLUSTER DISTRIBUTED FILE SYSTEM  **, they should have the  **SAME UID  **on  **ALL NODES  **in the CLUSTER.  
   *  This IS EASY if you are using a service like red hat's IdM or OpenLDAP for authentication, **but if you are not **, you will need to run this command for each user on each server in the cluster, to **make sure ALL USERS ARE IN THE "hadoop" GROUP  **(i.e. the container-executor.group): `useradd -g hadoop tom` .... 

* In the Ambari Dashboard, select the YARN service and then click the "Start-All" button. Note: Both stopping and starting the services can take some time.

** And thats it ! You have enabled Ambari Hadoop on Gluster ! **

Go get some ice cream.  Then, when you get back.... 

** YOU CAN TEST YOUR NEW HADOOP CLUSTER BY RUNNING TERAGEN !  **

* To test your cluster, open a terminal window and navigate to /usr/lib/hadoop. Then su to one of the configured allowed.system.users (such as tom) and submit a Hadoop Job:

`bin/hadoop jar /usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples-2.2.0.2.0.6.0-101.jar teragen 1000 in`

* Questions?  Feel free to ping the gluster mailing list: **gluster-users@gluster.org **.  
* Having fun with hadoop on gluster? Let us know  **@wattsteve @jayunit100 @gluster **on twitter :).

** Common Errors  **

* If you see an exception stating that "job.jar changed on src filesystem" it means that you need to synchronize the clocks across your cluster. You can do this by running the following command on each node:
`ntpd -qg`

------------------