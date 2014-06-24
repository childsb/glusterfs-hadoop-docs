**Directions for Running CDH5 Solr**

These instructions are experimental, currently.  

First edit properties in /etc/default/solr to match your cluster.  

    SOLR_HDFS_HOME=glusterfs:///solr 

Then, set up the services by running these commands: 

    /usr/lib/solr/tomcat-deployment.sh
    /etc/solr/conf/log4j.properties # <-- set properties . or just leave blank.
    /usr/lib/solr/bin/solrd run

This starts a basic SOLR server on port 8983.
From here, you will want to make sure you are plugged into the gluster file system for backend storage. 