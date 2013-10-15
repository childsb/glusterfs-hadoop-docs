GlusterFS is a clustered distributed file system that requires two or more servers. GlusterFS needs to be installed on each server within the cluster. After which a GlusterFS volume needs to be created and configured. 

The instructions below assume we are building a 4 node GlusterFS cluster using hostnames server1-4:

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

** Installing and Configuring GlusterFS **

1) On each server install and start GlusterFS on each server by doing the following:
`yum install glusterfs glusterfs-server glusterfs-fuse`
`service  glusterd start` 

If you are using Fedora 19, starting the service will fail due to a bug in Fedora. Just ignore the failure and additionally run the commands below:

`cd /usr/sbin`
`./glusterd`

Stop the Firewall so you can successfully peer probe
`service iptables stop`
`chkconfig iptables off`
`systemctl stop firewalld.service`

Create your brick. This can be a single directory or a block device you mount onto this directory
`mkdir /mnt/brick1`

2) Installing the FUSE Kernel Patch

There is presently a bug in FUSE which causes namespace consistency issues in GlusterFS. We have submitted an upstream patch and are working to ensure that future versions of the Linux Kernel will include it automatically.

In the interim, we have provided a workaround for RHEL 6.2 

RHEL 6 - Download the two RPMs required, [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-2.6.32-220.34.1.el6.test.x86_64.rpm) and [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm) 

Distribute the downloaded RPMs to each server in the cluster or ensure they are acessible via an NFS mount

Navigate to the directory with the RPMS and install them as follows:
`yum -y install kernel-2.6.32-220.34.1.el6.test.x86_64.rpm `
`yum -y install kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm`

3) On server-1, peer probe the other servers within the cluster to create a gluster trusted storage pool (this defines the GlusterFS "cluster") by running the following commands:
`gluster peer probe server-2`
`gluster peer probe server-3`
`gluster peer probe server-4`

4) On server-1, run the following commands to build the GlusterFS Volume:
`gluster volume create HadoopVol  server-1:/mnt/brick1/hadoop server-2:/mnt/brick1/hadoop server-3:/mnt/brick1/hadoop server-4:/mnt/brick1/hadoop `
`gluster volume start HadoopVol`
`gluster volume status`

5) On server-1, once the GlusterFS volume has been created, one needs to set the following appropriate parameters on the GlusterFS volume to ensure consistency across the namespace. This example assumes your GlusterFS volume is called "HadoopVol":

`gluster volume set HadoopVol quick-read off`
`gluster volume set HadoopVol cluster.eager-lock on`
`gluster volume set HadoopVol performance.stat-prefetch off`

6) Create the Mapred System Directory on the Gluster Volume Mount **

Open a terminal and run the following command:

`mkdir -p /mnt/glusterfs/mapred/system`
`chmod -R 2770 /mnt/glusterfs`

7) On each server, mount the Gluster volume to /mnt/glusterfs on every node within the trusted storage pool. Please note that this is a specialized mount command that sets the attribute and entry timeouts to zero. This is also required for namespace consistency in highly parallel environments. It is recommended that you take measures to ensure the mount is persisted upon reboot.

`glusterfs --attribute-timeout=0 --entry-timeout=0 --volfile-id=/HadoopVol --volfile-server=<HOST_NAME> /mnt/glusterfs`

