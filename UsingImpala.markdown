Note that currently, Impala doesn't yet support GlusterFS.  But you can set up the GlusterFileSystem to be the backing store for impala, and if you recompile bits of impala, you can get it to work, in theory.  

*For those really interested in running impala on alternate file systems, contact us directly and we are happy to discuss the details*

We have indeed managed to glue it in to CDH5, but unless a comprehensive set of changes is made to Impala (as of July 15, 2014), you cannot run it on non-HDFS file systems.


For the brave, however, here are our notes on configuring Impala.  You can actually get Impala to start up and read from the gluster filesystem, and even use impyla to create an impala table, using approximately the directions below.

Once we work to make Impala allow any HCFS implementation (Should be removing a few possibly unnecessary casting checks for HDFS), Impala should theoretically support any File system implementation as per the protocol below .

To set up impala , you first must set up hive in server mode.  That can be done following the directions in these two places:

[Installing Hive](http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hive_install.html)
[Configuring the Hive Metastore] (http://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_hive_metastore_configure.html)

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

4) 


5) Now, start impala services:

    sudo service impala-server restart ; 
    sudo service impala-state-store restart ; 
    sudo service impala-catalog restart;

And confirm that they are running:

    netstat -tupln | grep impalad

The above command should list several services in the 21000-213000 range.

The imapad service running on port 21050 is the one we will connect to using the python impyla client.

 

6) And install the impyla python client.  This client is more robust than the impala-shell app with respect

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


