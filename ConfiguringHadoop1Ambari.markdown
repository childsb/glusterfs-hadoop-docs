These instructions will provide you step-by-step instructions for using Ambari to configure your Hadoop 1.x Cluster on top of GlusterFS.
These instrctions assume you have already installed and configured GlusterFS per this link(https://forge.gluster.org/hadoop/pages/Configuration)

**Download the Ambari RPMS**

Navigate to [GlusterFS tool repository](http://hadoop.apache.org/releases.html#Download) and download the ambari-server-1.3.0-SNAPSHOT20130904172038.noarch.rpm and  ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm (to the /tmp directory on your cluster management node). 

Then copy the ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm to all the other nodes in your cluster.

**Pre-installation setup**
Once you have copied the rpms do the following on each node complete the following steps:

1. Navigate to the proper directory
    cd /tmp

2. Install the EPEL repo
    yum install epel-release

3. Install the ambari-server and ambari-agent rpms
    yum install ambari-server-1.3.0-SNAPSHOT20130904172038.noarch.rpm
    yum install ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm

4. On the server node run the setup program:
    ambari-server setup -s _(this will install the server with the current defaults)_

5. On the agent nodes (including the server node) configure the Ambari Agent by editing the ambari-agent.ini file:
   vi  /etc/ambari-agent/conf/ambari-agent.ini
          [server]
          hostname={your.ambari.server.hostname} <------EDIT THIS LINE
          url_port=8440
          secured_url_port=8441

6. Start the Ambari Server
    ambari-server start

7. Start the Ambari Agent
    ambari-agent start
