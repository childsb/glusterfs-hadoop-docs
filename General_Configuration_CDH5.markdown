##On Each Node

1) Get the CDH5 repo: 
    yum-config-manager --add-repo http://archive.cloudera.com/cdh5/redhat/5/x86_64/cdh/cloudera-cdh5.repo
    
2) Then install it: 
    yum-config-manager --enable cloudera-cdh5

3) Install the hadoop contents on your system: 
    yum install hadoop hadoop-mapreduce hadoop-yarn

4) Install jdk-devel packages.
    yum install -y java-<$VERSION>-openjdk-devel.x86_64

5) Change the group ownership of the container-executor and reset the permissions.
    chgrp hadoop /usr/lib/hadoop-yarn/bin/container-executor
    chmod 6050 /usr/lib/hadoop-yarn/bin/container-executor
##On Head Node

_All *.xml files below can be found in `/etc/hadoop/conf/`.  For brevity, we specify them as key = value pairs._

e.g.
    <property>
        <name>I am a key</name>
        <value>And I am the value</value>
    </property>
1) Edit **core-site.xml**.  

* `fs.glusterfs.impl = org.apache.hadoop.fs.glusterfs.GlusterFileSystem` 
* `fs.default.name = glusterfs:///` 
* `fs.glusterfs.mount = /mnt/glusterfs`
* `fs.AbstractFileSystem.glusterfs.impl = org.apache.hadoop.fs.local.GlusterFs`
* `fs.glusterfs.volumes = HadoopVol`
* `fs.glusterfs.volume.fuse.HadoopVol = /mnt/glusterfs`


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

##Synchronize Config Settings

1) Copy core-site.xml, yarn-site.xml, and mapred-site.xml to each node on the cluster:

    scp core-site.xml yarn-site.xml mapred-site.xml root@$NODE:/etc/hadoop/conf/

2) On each **slave** node, edit the following properties in **yarn-site.xml**. Do not edit master's yarn-site.xml.

* yarn.nodemanager.hostname = $LOCAL_IP
* yarn.resourcemanager.hostname = $MASTER_IP

3) Sync server clocks.  On each server, run the following. Creating a crontab command of it for every minute is also a good idea.
    
    ntpd -qg

##Again, On Each Node

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

##Startup

1) Set the permissions on your container logging directory (i.e. /var/log/hadoop-yarn/containers/ ) so that yarn OWNS it.  This is essential for running a mapreduce job.  Otherwise, your jobs can hang.  

2) Set permissions for yarn log directory:
        chown yarn /usr/lib/hadoop-yarn/

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

   

##Run a MapReduce Job

Switch to the mapreduce designated user (tom), and run a mapred job.

    su tom
    cd /usr/lib/hadoop
    bin/hadoop jar ../hadoop-mapreduce/hadoop-mapreduce-examples.jar pi 1 1
