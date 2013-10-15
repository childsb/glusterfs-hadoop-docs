**Note: These instructions will only work on RHEL. Apache Ambari does not yet work on Fedora **

These instructions will provide you step-by-step instructions for using Ambari to configure your Hadoop 1.x Cluster on top of GlusterFS.

These instructions assume you have already [installed and configured GlusterFS](https://forge.gluster.org/hadoop/pages/Configuration)

**Download the Ambari RPMS**

Use the following links to dowload the Ambari RPMs:
[Ambari-server](https://s3-us-west-1.amazonaws.com/rhbd/glusterfs-ambari/ambari-server-1.3.0-SNAPSHOT20130904172038.noarch.rpm)
[Ambari-agent](https://s3-us-west-1.amazonaws.com/rhbd/glusterfs-ambari/ambari-agent-1.3.0-SNAPSHOT20130904172112.x86_64.rpm) 
to the /tmp directory on your cluster management node.

Then copy the Ambari-agent RPM to all the other nodes in your cluster.

**Pre-installation setup**

Once you have copied the rpms do the following on each node complete the following steps:

0. Set permissions for the getfattr process
    vi /etc/sudoers.d/gluster
    mapred ALL= NOPASSWD: /usr/bin/getfattr
    chmod 440 /etc/sudoers.d/gluster

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
          hostname={your.ambari.server.hostname}
          url_port=8440
          secured_url_port=8441

6. Start the Ambari Server
    ambari-server start

7. Start the Ambari Agent
    ambari-agent start

**Ambari wizard setup**
Once you have started the Ambari-server and Ambari agent open a browser to:
http://your.ambari.server.hostname:8080

Once the login screen loads use admin/admin for the credentials and press the Sign In button.

1. Welcome: Assign a name to your Cluster
2. Select Stack: Select the HDP 1.3.2-GlusterFS Stack
3. Install Options: 
	--Target Hosts: Enter one host per line including the hosts used for the install if it is part of the Storage Pool. 
	--Host Registration Information: Select ‘Perform 	Manual Registration on hosts, do not use SSH’.
	(You will get a pop of warning. The manual install of the agents was completed by the install.sh script.  No additional action needs to take place.  Select OK and continue)

	--Advanced Options: Do not select any values in this section. 
          Select Next to continue to the following screen.  
          (You will receive another pop-up about manually registering 
          your Ambari-agents.  This has already been completed, select OK
          to continue.)

4. Confirm Hosts: Confirm all the hosts you entered from the previous page are present. The system will now register the hosts in the cluster.  Select Next once the registration process is complete.

5. Choose Services: First press the minimum link to remove any unnecessary services. Then select GlusterFS and MapReduce.  Then click next.  Note, if you do not select Nagios and/or Ganglia you will receive a pop-up warning of limited functionality.  Select OK to continue.

6. Assign Masters: Chose which services you want to run on each host.  Select Next when finished

7. Assign Slaves and Clients: Chose which components you want on each hosts. Select the check box to put the GlusterFS client on all the nodes.

8. Customize Services: For the GlusterFS tab you can accept the defaults. If you have chosen to install Nagios you will need to enter a password and email for alerts.  Under the MapReduce tab set the value for mapred.local.dir to /mnt/brick1/mapredlocal.  Select Next to continue.

9. Review: Make sure the services are what you selected in the previous steps. Select Deploy to continue.  You are done once the deploy process completes!