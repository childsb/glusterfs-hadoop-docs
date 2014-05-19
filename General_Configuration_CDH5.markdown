Generic Configuration instructions 

1) Install into your /etc/yum.repos.d/ the CDH5 yum repos .

Get the repo: 
    
    yum-config-manager --add-repo http://archive.cloudera.com/cdh5/redhat/5/x86_64/cdh/cloudera-cdh5.repo
    
and then install it: 

    yum-config-manager --enable cloudera-cdh5

2) Now, yum install the hadoop contents on your system: 

    yum install hadoop
    yum install hadoop-mapreduce
    yum install hadoop-yarn
    
3) Now add these properties to your core-site.xml 

```
<property>
    <name>fs.glusterfs.impl</name>
    <value>org.apache.hadoop.fs.glusterfs.GlusterFileSystem</value>
  </property>
  <property>
    <name>fs.default.name</name>
    <value>glusterfs:///</value>
  </property>

  <property>
   <name>fs.glusterfs.mount</name>
   <value>/mnt/glusterfs</value>
   </property>

  <property>
    <name>fs.AbstractFileSystem.glusterfs.impl</name>
    <value>org.apache.hadoop.fs.local.GlusterFs</value>
  </property>

    <property>
    <name>fs.AbstractFileSystem.glusterfs.impl</name>
    <value>org.apache.hadoop.fs.local.GlusterFsCRC</value>
  </property>
```

4) Now edit your yarn-site.xml file


```
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
<property>
   <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
   <value>org.apache.hadoop.mapred.ShuffleHandler</value></property>
<property>
   <name>yarn.log-aggregation-enable</name>
   <value>true</value>
</property>
<property>
   <name>yarn.nodemanager.local-dirs</name>
   <value>/var/lib/hadoop-yarn/cache/${user.name}/nm-local-dir</value>
</property>
<property>
   <name>yarn.nodemanager.log-dirs</name>
   <value>/var/log/hadoop-yarn/containers</value>
</property>
<property>
<description>Where to aggregate logs to.</description>
   <name>yarn.nodemanager.remote-app-log-dir</name>
   <value>glusterfs:///var/log/hadoop-yarn/apps</value>
</property>
<property>
   <name>yarn.application.classpath</name>
   <value>$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,$HADOOP_YARN_HOME/share/hadoop/yarn/*,$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*</value>
</property>
<property>
   <name>mapreduce.jobtracker.address</name>
   <value>localhost</value>
</property>

```

5) Make sure all your hadoop libraries are on the classpath.  A good way to do this is to simply hardcode them, into the mapreduce.application.classpath parameter.  This prevents reliance on environmental variables and is easier to debug.  

```
   <name>mapreduce.application.classpath</name>
   <value>/usr/lib/hadoop-yarn/lib/*,/usr/lib/hadoop-yarn/*,/usr/lib/hadoop/lib/*,/usr/lib/hadoop/*,/usr/lib/hadoop-mapreduce/*,$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,$HADOOP_MAPRED_HOME/lib/*</value>
```

6) And then add this property to your mapred-site.xml as well.

```
<name>yarn.app.mapreduce.am.staging-dir</name>
<value>glusterfs:///tmp/hadoop-yarn/staging</value>

```

7) Update your remote logging directory to write logs to glusterfs:///.  


```
<property>
    <description>Where to aggregate logs to.</description>
    <name>yarn.nodemanager.remote-app-log-dir</name>
    <value>glusterfs:///var/log/hadoop-yarn/apps</value>
  </property>
```

Later, if you need to debug a job, yarn will be able to pull the logs up for you using this command (replace the 1234... number with the application id of your job).

`/usr/lib/hadoop-yarn/bin/yarn --applicationId 12345678_1234 `

8) 

```
 <name>yarn.app.mapreduce.am.staging-dir</name>
 <value>glusterfs:///tmp/hadoop-yarn/staging</value>
```

9) Thats it ! Now you can run users, albeit just as the yarn user, in CDH5.  