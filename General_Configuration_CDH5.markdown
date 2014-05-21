## On each machine ## 

1) Get the CDH5 repo: `yum-config-manager --add-repo http://archive.cloudera.com/cdh5/redhat/5/x86_64/cdh/cloudera-cdh5.repo`
    
2) and then install it: `yum-config-manager --enable cloudera-cdh5`

3) Now, yum install the hadoop contents on your system: `yum install hadoop hadoop-mapreduce hadoop-yarn`

4) Install jdk-devel packages.  For example `yum install -y java-1.6.0-openjdk-devel.x86_64`
    
## On head node ##

3) Now add these properties to your core-site.xml.  For brevity, we specify them as key=value pairs. 

`fs.glusterfs.impl=org.apache.hadoop.fs.glusterfs.GlusterFileSystem` 
`fs.default.name=glusterfs:///` 
`fs.glusterfs.mount=/mnt/glusterfs` 
`fs.AbstractFileSystem.glusterfs.impl=org.apache.hadoop.fs.local.GlusterFs` 

4) Now edit your yarn-site.xml file.  Note that the "**MASTER**" value at the bottom needs to be the IP of your master node.

`yarn.nodemanager.aux-services=mapreduce_shuffle` 
`yarn.nodemanager.aux-services.mapreduce_shuffle.class=org.apache.hadoop.mapred.ShuffleHandler`
`yarn.log-aggregation-enable=true` 
`yarn.nodemanager.local-dirs=/var/lib/hadoop-yarn/cache/${user.name}/nm-local-dir` 
`yarn.nodemanager.log-dirs=/var/log/hadoop-yarn/containers` 
`yarn.nodemanager.remote-app-log-dir=glusterfs:///var/log/hadoop-yarn/apps` 
`yarn.application.classpath=/usr/lib/hadoop-yarn/lib/*,/usr/lib/hadoop-yarn/*,/usr/lib/hadoop/lib/*,/usr/lib/hadoop/*,/usr/lib/hadoop-mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/lib/*,$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,$HADOOP_YARN_HOME/share/hadoop/yarn/*,$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*</value>
mapreduce.jobtracker.address=**MASTER**`

Now, in your mapred-site.xml file:

5.0) Set/add the xml property "mapreduce.framework.name", and set it equal to "yarn".

5.1) Make sure all your hadoop libraries are on the classpath.  A good way to do this is to simply hardcode them, into the mapreduce.application.classpath parameter.  This prevents reliance on environmental variables and is easier to debug.  

`mapreduce.application.classpath=/usr/lib/hadoop-yarn/lib/*,/usr/lib/hadoop-yarn/*,/usr/lib/hadoop/lib/*,/usr/lib/hadoop/*,/usr/lib/hadoop-mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/lib/*`

6) And then add this property to your mapred-site.xml as well.

`<name></namyarn.app.mapreduce.am.staging-dire>
<value>glusterfs:///tmp/hadoop-yarn/staging</value>`

7) Update your remote logging directory to write logs to glusterfs:///.  

`<property>
    <description>Where to aggregate logs to.</description>
    <name>yarn.nodemanager.remote-app-log-dir</name>
    <value>glusterfs:///var/log/hadoop-yarn/apps</value>
  </property>`

Later, if you need to debug a job, yarn will be able to pull the logs up for you using this command (replace the 1234... number with the application id of your job) : `/usr/lib/hadoop-yarn/bin/yarn --applicationId 12345678_1234 `

8) Update your staging directory in yarn-site.xml

 `yarn.app.mapreduce.am.staging-dir=glusterfs:///tmp/hadoop-yarn/staging</value>` 

9) Thats it ! Now you can run users, albeit just as the yarn user, in CDH5.  

## SYNCHRONIZE CONFIGURATION SETTINGS

Copy your core-site.xml, yarn-site.xml, and mapred-site.xml files to each node on the cluster.

And finally, on each node, make sure "yarn.nodemanager.hostname" points to the IP of your master node.

## Saving hadoop Evn variables

Now save this snippet in /opt/env.sh, with executable permissions (make sure you modify the JAVA_HOME to match your java distro).

   (some might require fixing... TODO)
    export HADOOP_CONF_DIR=/etc/hadoop/conf
    export JAVA_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64/ 
    export HADOOP_LIBEXEC_DIR=/usr/lib/hadoop/libexec
    export HADOOP_YARN_HOME=/usr/lib/hadoop-yarn/
    export HADOOP_MAPRED_HOME=/usr/lib/hadoop-mapreduce/
    export YARN_HOME=/usr/lib/hadoop-yarn/

We will reference it at other times. 

##  STARTUP 

0)  Set the permissions on your container logging directory (i.e. /var/log/hadoop-yarn/containers/ ) so that yarn can write to it.

1) su to user "yarn".  This user is created for you when you yum install cloudera hadoop. 

2) You can now restart all your hadoop services.   A simple snippet to do this follows:


    chown yarn /usr/lib/hadoop-yarn/  
    killall -9 java
    export JAVA_HOME=/usr/lib/jvm/jre-1.7.0-openjdk.x86_64/ 
    export HADOOP_LIBEXEC_DIR=/usr/lib/hadoop/libexec
    export HADOOP_COMMON_HOME=/usr/lib/hadoop/
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh stop nodemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh stop resourcemanager 
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start resourcemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start nodemanager 

