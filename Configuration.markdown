## Solution Components ##

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* GlusterFS 3.3 +
* FUSE Kernel Patch
* Apache Hadoop 1.x or 2.x
* The GlusterFS Hadoop FileSystem Plugin
* Oracle Java Runtime Environment (JRE) 1.6 +
* NTP

## Configuration Guide ##

This guide is focused on giving the community a way to quickly evaluate Hadoop on GlusterFS. When our work on the Apache Ambari project is complete, we will be able to automate all of this via an installer. A popular way to try the solution out is to set up 4 Virtual Machines with RHEL 6.x and then follow the instructions below.

** Installing the FUSE Kernel Patch **

There is presently a bug in FUSE which causes namespace consistency issues in GlusterFS. We have submitted an upstream patch and are working to ensure that  future versions of the Linux Kernel will include it automatically.

In the interim, we have provided a workaround for RHEL 6.x and Centos 6.x kernels.  This workaround does not work with Fedora. 

* Please download the two RPMs required, [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-2.6.32-220.34.1.el6.test.x86_64.rpm) and [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm)
* Distribute the downloaded RPMs to each server in the cluster or ensure they are acessible via an NFS mount
* Navigate to the directory with the RPMS and install them as follows:

`yum -y install kernel-2.6.32-220.34.1.el6.test.x86_64.rpm` 
`yum -y install kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm`


** Installing and Configure GlusterFS** 

Install and configure GlusterFS on a cluster of servers. Build your trusted storage pool and create your gluster volume. The majority of testing with Hadoop has been done on 'Distributed' and 'Distributed Replicated 2' volume types. 
Gluster Documentation and Downloads [are available here](http://www.gluster.org/download/)

** Install Oracle Java 1.6 **

This is a requirement of Hadoop. Hadoop has not yet been widely tested with OpenJDK, although it may well work. The JRE needs to be installed on every server within the trusted storage pool that your Gluster Volume uses.

** Create the Mapred System Directory on the Gluster Volume Mount **

Open a terminal and run the following command:

`mkdir -p /mnt/glusterfs/mapred/system`
`chmod -R 1777 /mnt/glusterfs`

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

**For Automated Hadoop 1.x Installation:** please see - [Automated Hadoop 1.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop1Ambari) for GlusterFS

**For Hadoop 2.x:** please see - [Configuring Hadoop 2.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop2) for GlusterFS