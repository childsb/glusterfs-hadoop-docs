## Solution Components ##

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* GlusterFS 3.3 +
* FUSE Kernel Patch
* Apache Hadoop 1.x or 2.x
* The GlusterFS Hadoop FileSystem Plugin
* Oracle Java Runtime Environment (JRE) 1.6 +
* NTP

## Configuration Guide ##

This guide is focused on giving the community a way to quickly evaluate Hadoop on GlusterFS. When our work on the Apache Ambari project is complete, we will be able to automate all of this via an installer. A popular way to try the solution out is to set up 4 Virtual Machines with RHEL 6.2  or Fedora 19 and then follow the instructions below.

** Installing and Configure GlusterFS** 

Prior to Installing and Configuring Hadoop, once needs to first install GlusterFS and configure a GlusterFS volume. [Follow these instructions here to achieve this](https://forge.gluster.org/hadoop/pages/InstallingAndConfiguringGlusterFS).

The majority of testing with Hadoop has been done on 'Distributed' and 'Distributed Replicated 2' GlusterFS volume types. Additional Gluster Documentation and Downloads [are available here](http://www.gluster.org/download/)

** Install Oracle Java 1.6 **

This is a requirement of Hadoop. Hadoop has not yet been widely tested with OpenJDK, although it may well work. The JRE needs to be installed on every server within the trusted storage pool that your Gluster Volume uses.

** Create the Mapred System Directory on the Gluster Volume Mount **

Open a terminal and run the following command:

`mkdir -p /mnt/glusterfs/mapred/system`
`chmod -R 2770 /mnt/glusterfs`

**Configure Passwordless SSH**

Designate a server within your trusted storage pool to run the JobTracker for Hadoop 1.0 or the Resource Manager in the case of Hadoop 2.0 . For the SSH instructions, we will call this server the Master Server. We will set up passwordless SSH from the Master server to all the other nodes in the cluster.


On the Master Server, run the following command:

     ssh-keygen

     (Hit Enter to accept all of the defaults)

On the Master Server, run the following command for each server. Server, run the following command for each server.

     ssh-copy-id -i ~/.ssh/id_rsa.pub root@<hostname>

For example, if you had four servers in your cluster with the hostnames svr1, svr2, svr3 and svr4 and svr1 is your  Master Server, then you would run the following commands from svr1 after you had run ssh-keygen:

     ssh-copy-id -i ~/.ssh/id_rsa.pub root@svr1
     ssh-copy-id -i ~/.ssh/id_rsa.pub root@svr2
     ssh-copy-id -i ~/.ssh/id_rsa.pub root@svr3
     ssh-copy-id -i ~/.ssh/id_rsa.pub root@svr4
    
Lastly, verify you can ssh from the Master Server to all the other servers without being prompted for a password.

**Sync Clocks**

It is important that the time and date be synchronized for each server within the cluster.   The Network Time Protocol (NTP) is a popular method to achieve this. 

To install NTP on each node run:

     yum install ntp

To update the clock on each machine run:
     
    ntpd -qg

Consult the ntpd documentation to configure periodic time resync.

** Configuring Hadoop **

**For Hadoop 1.x Manual Installation:** please see - [Configuring Hadoop 1.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop1) for GlusterFS

**For Hadoop 2.x:** please see - [Configuring Hadoop 2.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop2) for GlusterFS