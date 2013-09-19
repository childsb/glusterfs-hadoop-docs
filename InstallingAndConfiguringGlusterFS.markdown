GlusterFS is a clustered distributed file system that requires two or more servers. GlusterFS needs to be installed on each server within the cluster. After which a GlusterFS volume needs to be created and configured. 

To achieve this on Fedora 19, please follow the instructions below. These instructions assume we are building a 4 node GlusterFS cluster using hostnames server1-4:

1) On each server, [download and Install Fedora 19](http://fedoraproject.org/en/get-fedora) 

2) On each server install and start GlusterFS on each server by doing the following:
`yum install glusterfs glusterfs-server glusterfs-fuse`
`service  glusterd start` 

(starting the service will fail due to a bug in Fedora, but its required to get glusterd into the /usr/sbin directory. Just ignore the failure)

`cd /usr/sbin`
`./glusterd`

Stop the Firewall so you can successfully peer probe
`service iptables stop`
`chkconfig iptables off`
`systemctl stop firewalld.service`

Create your brick. This can a single directory or a block device you mount onto this directory
`mkdir /mnt/brick1`

3) Installing the FUSE Kernel Patch

There is presently a bug in FUSE which causes namespace consistency issues in GlusterFS. We have submitted an upstream patch and are working to ensure that future versions of the Linux Kernel will include it automatically.

In the interim, we have provided a workaround for RHEL 6.2 and Fedora 19 

RHEL 6 - Download the two RPMs required, [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-2.6.32-220.34.1.el6.test.x86_64.rpm) and [here](http://rhbd.s3.amazonaws.com/glusterfs-hadoop/kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm) 
Fedora 19 - Download the two RPMs required here and here 

Distribute the downloaded RPMs to each server in the cluster or ensure they are acessible via an NFS mount

Navigate to the directory with the RPMS and install them as follows:
`yum -y install kernel-2.6.32-220.34.1.el6.test.x86_64.rpm `
`yum -y install kernel-firmware-2.6.32-220.34.1.el6.test.noarch.rpm`

4) On server-1, peer probe the other servers within the cluster to create a gluster trusted storage pool (this defines the GlusterFS "cluster") by running the following commands:
`gluster peer probe server-2`
`gluster peer probe server-3`
`gluster peer probe server-4`

5) On server-1, run the following commands to build the GlusterFS Volume:
`gluster volume create HadoopVol  server-1:/mnt/brick1/hadoop server-2:/mnt/brick1/hadoop server-3:/mnt/brick1/hadoop server-4:/mnt/brick1/hadoop `
`gluster volume start HadoopVol`
`gluster volume status`

6) On server-1, once the GlusterFS volume has been created, one needs to set the following appropriate parameters on the GlusterFS volume to ensure consistency across the namespace. This example assumes your GlusterFS volume is called "HadoopVol":

`gluster volume set HadoopVol quick-read off`
`gluster volume set HadoopVol cluster.eager-lock on`
`gluster volume set HadoopVol performance.stat-prefetch off`

7) On each server, mount the Gluster volume to /mnt/glusterfs on every node within the trusted storage pool. Please note that this is a specialized mount command that sets the attribute and entry timeouts to zero. This is also required for namespace consistency in highly parallel environments. It is recommended that you take measures to ensure the mount is persisted upon reboot.

`glusterfs --attribute-timeout=0 --entry-timeout=0 --volfile-id=/HadoopVol --volfile-server=<HOST_NAME> /mnt/glusterfs`

