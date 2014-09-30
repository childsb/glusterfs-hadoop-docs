# Setting up CDH5 #

All the apache components of CDH5, to our knowledge, are 100% HCFS compatible on glusterfs-hadoop from the basic testing that we've done.  Here is how to get started with a Cloudera's recent CDH5 release on hadoop.


**For CDH5: Simple and Secure Modes ** 
See - [CDH5 (simple) ] (https://forge.gluster.org/hadoop/pages/Cloudera#Setting+up+Linux+Container+Executor+-+(Unsecured)) for GlusterFS

**For CDH 5: Full security mode ** 
See - [CDH5 (kerberos) ] (https://forge.gluster.org/hadoop/pages/Cloudera#Secureing+CHD5+Hadoop+with+Kerberos) for GlusterFS

# Setting up the Hadoop Ecosystem on top of CDH5 #

Common hadoop client applications (*Pig, Hive, Flume, Sqoop, Zookeeper, and Mahout*) all run as is, as hadoop clients, with no extra configuration required.  *Meanwhile*, the services which run on hadoop (i.e. *Oozie, HBase, Solr*  ) require more sohpisticated configuration steps, because you need to update certain files, and possibly add the glusterfs-hadoop plugin into directories other than the /usr/lib/hadoop/lib location.

So lets get started.

## Part 1: Hadoop Client apps on glusterfs ##

After you've yum installed the client apps (`yum install hive pig mahout flume-ng zookeeper sqoop`) you can use [Our backported BigTop Smoke Tests] (https://github.com/roofmonkey/simple-smokes/) to test them quite easily.  You will have to export some variables first.

    export HIVE_CONF_DIR=/etc/hive/conf/
    export PIG_HOME=/usr/lib/pig/
    export SQOOP_HOME=/usr/lib/sqoop/

And then you can run the test.sh file in the top directory.

## Part 2: Hadoop Service applications on glusterfs ##

We have specific directions for installing and running CDH5 ecosystem service applications below.

* [HBase](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+HBase). This page also includes instructions how to test Hbase.

* [Oozie](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+Oozie).  For testing Oozie, you can use https://forge.gluster.org/hadoop/oozie-smoke-test.

* [Solr](https://forge.gluster.org/hadoop/pages/AdditionalComponents#Using+Solr).  This page has specific instructions on how to test Solr. 

If you have any issues, you can contact us directly or ping the gluster users mailing list. 


# Start Here: Cloudera Hadoop on GlusterFS #

##Overview.

First, we'll set up hadoop on gluster, with no security, using the yarn container executor (which is the default).

Then we will add the GlusterContainer Executor in as a quick and simple way to run jobs on CDH5 against glusterfs with multitenancy.

##Part 1: On Each Node

1) Get the CDH5 repo: 
    yum-config-manager --add-repo http://archive.cloudera.com/cdh5/redhat/5/x86_64/cdh/cloudera-cdh5.repo
    
2) Then install it: 
    yum-config-manager --enable cloudera-cdh5

3) Install the hadoop contents on your system: 
    yum install hadoop hadoop-mapreduce hadoop-yarn

4) Install jdk-devel packages.
    yum install -y java-<$VERSION>-openjdk-devel.x86_64

5) Change the group ownership of the container-executor and reset the permissions.  Make sure the user ownership is root.
    chgrp hadoop /usr/lib/hadoop-yarn/bin/container-executor
    chmod 6050 /usr/lib/hadoop-yarn/bin/container-executor
##Part 2: On Head Node

_All *.xml files below can be found in `/etc/hadoop/conf/`.  For brevity, we specify them as key = value pairs._

e.g.
    <property>
        <name>I am a key</name>
        <value>And I am the value</value>
    </property>
1) Edit **core-site.xml**

* fs.glusterfs.impl = org.apache.hadoop.fs.glusterfs.GlusterFileSystem
* fs.default.name = glusterfs:///
* fs.glusterfs.mount = /mnt/glusterfs
* fs.AbstractFileSystem.glusterfs.impl = org.apache.hadoop.fs.local.GlusterFs
* fs.glusterfs.volumes = HadoopVol
* fs.glusterfs.volume.fuse.HadoopVol = /mnt/glusterfs


2) Edit **yarn-site.xml** .

Update the yarn class path:

* `yarn.application.classpath = /usr/lib/hadoop-yarn/lib/*,/usr/lib/hadoop-yarn/*,/usr/lib/hadoop/lib/*,/usr/lib/hadoop/*,/usr/lib/hadoop-mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/lib/*,$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,$HADOOP_YARN_HOME/share/hadoop/yarn/*,$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*`

Add:

* `mapreduce.jobtracker.address =` **$MASTER_IP**
* `yarn.app.mapreduce.am.staging-dir = glusterfs:///tmp/hadoop-yarn/staging`

3) Edit  **mapred-site.xml**:

Make sure all your hadoop libraries are in the classpath property.  They can be hard-coded into `mapreduce.application.classpath` to remove reliance on environmental variables and ease debugging.

Add / Set the following properties:

* `mapred.system.dir = glusterfs:///mapred/system`
* `mapred.jobtracker.system.dir = glusterfs:///mapred/system`
* `mapreduce.framework.name = yarn`
* `mapreduce.jobtracker.system.dir = glusterfs:///mapred/system`
* `mapreduce.application.classpath = /usr/lib/hadoop-yarn/lib/*,/usr/lib/hadoop-yarn/*,/usr/lib/hadoop/lib/*,/usr/lib/hadoop/*,/usr/lib/hadoop-mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/lib/*`

Just as in the yarn-site.xml, update staging directory:

* `yarn.app.mapreduce.am.staging-dir = glusterfs:///tmp/hadoop-yarn/staging`

Create a mapred/system/ dir and set it's privileges.
    mkdir -p /mnt/glusterfs/mapred/system
    chown mapred:hadoop /mnt/glusterfs/mapred/system

##Part 3: Synchronize Config Settings

1) Copy core-site.xml, yarn-site.xml, and mapred-site.xml to each node on the cluster:

    scp core-site.xml yarn-site.xml mapred-site.xml root@$NODE:/etc/hadoop/conf/

2) On each **slave** node, edit the following properties in **yarn-site.xml**. Do not edit master's yarn-site.xml.

* yarn.nodemanager.hostname = $LOCAL_IP
* yarn.resourcemanager.hostname = $MASTER_IP

3) Sync server clocks.  On each server, run the following. Creating a crontab command of it for every minute is also a good idea.
    
    ntpd -qg

4) Copy the plugin into the /lib directory of hadoop, by running the following command.

`wget http://rhbd.s3.amazonaws.com/maven/repositories/internal/org/apache/hadoop/fs/glusterfs/glusterfs-hadoop/2.3.5/glusterfs-hadoop-2.3.5.jar -O /usr/lib/hadoop/lib/glusterfs-hadoop-2.3.5.jar`

##Part 4: Again, On Each Node

1) Set Hadoop Environmental Variables

Create a shell script called `/etc/profile.d/hadoop_env.sh`. Be sure to set JAVA_HOME $VERSION to the installed version.

    export HADOOP_CONF_DIR=/etc/hadoop/conf
    export JAVA_HOME=/usr/lib/jvm/jre-<$VERSION>-openjdk.x86_64/ 
    export HADOOP_LIBEXEC_DIR=/usr/lib/hadoop/libexec
    export HADOOP_YARN_HOME=/usr/lib/hadoop-yarn/
    export HADOOP_MAPRED_HOME=/usr/lib/hadoop-mapreduce/
    export YARN_HOME=/usr/lib/hadoop-yarn/


2) On each node run as root:
    source /etc/profile.d/hadoop_env.sh

We will reference it at other times. 

##Part 5: Startup

1) Set the permissions on your container logging directory (i.e. /var/log/hadoop-yarn/containers/ ) so that yarn OWNS it.  This is essential for running a mapreduce job.  Otherwise, your jobs can hang.

2) Set permissions for yarn log directory:
        chown yarn /usr/lib/hadoop-yarn/logs

_On VMs Only_
    Set yarn-site.xml yarn.scheduler.minimum-allocation-mb to a low enough value (i.e. ~ 1/2 of allocated VM memory). 

3) su to user "yarn".  This user was created when you installed Cloudera Hadoop. 

4) You can now restart all your hadoop services.  Make sure environmental variables have been set.   A simple snippet that can be copied to script follows:

_Important!:_ Only start the resource manager on the master node.  Omit `resourcemanager` commands for all slave nodes.


    killall -9 java
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh stop nodemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh stop resourcemanager 
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start resourcemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start nodemanager


---------------------------
# CDH5 Simpe: Setting up Linux Container Executor #

##Part 1: Check your Yarn services.

If you have not already done so, confirm that you can start the yarn services. Export environment variables and then start nodemanager and resourcemanager services.  

_Important!:_ Only start the resource manager on the master node. Omit resourcemanager commands for all slave nodes.

_Note:_ `source hadoop_env.sh` is only necessary if hadoop_env.sh is not located in /etc/profile.d/.

    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start resourcemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start nodemanager

Then check for their java processes as yarn user.

    su yarn -c "jps"

Should return something similar to this (omiting ResourceManager on all slaves):

    <$LVMID>  NodeManager
    <$LVMID>  ResourceManager
    <$LVMID>  Jps


##Part 2: Running Hadoop with the GlusterContainerExecutor

_NOTE:_ Do 1 - 3 on **each** node.

1) Add/modify the following properties in the yarn-site.xml file.

* yarn.nodemanager.container-executor.class = org.apache.hadoop.yarn.server.nodemanager.GlusterContainerExecutor
* yarn.nodemanager.linux-container-executor.group = hadoop

2) Edit /etc/hadoop/conf/container-executor.cfg to mirror the following

    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom

3) Create user "tom" if it does not already exist on master node.

    useradd -u 1024 -g hadoop tom

4)  All hadoop users must have a UID of or greater than the value of `min.user.id` (1000) in the container-exectuor.cfg.  Any **one** of the following are appropriate ways to ensure identical UID/GID across the cluster.

* 4.1 ) Copy /etc/passwd and /etc/group from your head node to all others

* 4.2) Follow the IPA based user setup section of  [[Setting+Up+Cloudera#Kerberos:+Securing+CHD5+Hadoop+with+Kerberos]] OR 

* 4.3) Use your companies internal LDAP servers to provision system ids for you.


5) Restart your yarn and nodemanager services.  To do this, you can follow the snippet in the [[SettingUpCloudera#Part+5:+Startup]] section of 
General_Configuration_CDH5.

## Part 3: _Run Hadoop Jobs!_

To run mapreduce jobs, switch to the mapreduce designated user (tom) and run the following.

    su tom
    cd /usr/lib/hadoop/
    bin/hadoop jar ../hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 1 1

--------------------------------------------


#CHD5 Sercure: Kerberos #

##Part 1: Stop Everything

* Stop all hadoop services.  You can do this quickly with `killall -9 java` or you can find pids for the NodeManager and ResourceManager processes running  "jps" (do this as root, so you're gauranteed to see all of them), and kill them directly.

* Now, edit yarn-site.xml to contain following properties:

    yarn.nodemanager.container-executor.class=     org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor

    yarn.nodemanager.linux-container-executor.group=
        hadoop

##Part 2: Set up IPA 

- On your **Head Node**, `yum install ipa-server.`

- On **Each Client**, run `ipa-client-install`.  Enter in the master name as the KDC server when doing this. 

-  Add service principals on the head node using ipa-service-add for resource manager and node manager.

-  Add the hadoop group, the yarn group using `ipa group-add`.

-  Add members using `ipa group-add-member` for each user who will be running hadoop jobs on your cluster.   For example, user tom would be added using ipa.  

_Note:_ Since you need a resourcemanager and nodemanager principal, use "rm" / "nm" as your principals.

-  Now, get keytabs for each of the yarn and nodemanager services, and write them to a temp file.  We'll move it to /etc later.

-  Copy those keytabs to a directory on your local file system for each node.  

-  Configure your hadoop cluster with kerberos key tabs, updating the yarn-site.xml and core-site.xml the standard method.

_Note_: In this example, we reused keytab/service name for the first machine on all other nodes of cluster.  That is a bit of a compromise in terms of security. It means that if someone intercepts the keytab, all machines are compromised with respect to that particular service.

##Part 3: Follow Standard Kerberos Setup

On your **core-site.xml**:

* set hadoop.security.authentication=true
* set hadoop.security.authorization=true
* Add the following entry: 
    <name>hadoop.security.auth_to_local</name>
        <value>
            RULE:[1:$1@$0](.*@YOUR_REALM)s/@.*//
            DEFAULT
        </value>

On your **yarn-site.xml**: 

* set yarn.resourcemanager.keytab=/etc/hadoop/conf/rm.keytab
* yarn.nodemanager.keytab=nm/YOUR_HEAD_NODE@YOUR_REALM
* yarn.resourcemanager.keytab=rm/YOUR_HEAD_NODE@YOUR_REALM

##Part 4: Set user passwords for users

You can set hadoop users' (e.g. tom) passwords through the free ipa web ui : http://www.freeipa.org/page/Web_UI.  

##Part 5: Log in a user and run a job

Now, you will want to run "kinit yarn", and restart all services.  

Then use kerberos to login a hadoop user, i.e. `kinit tom`... and enter tom's password. 

