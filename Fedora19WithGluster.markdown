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

3) On server-1, peer probe the other servers within the cluster to create a gluster trusted storage pool (this defines the GlusterFS "cluster") by running the following commands:
`gluster peer probe server-2`
`gluster peer probe server-3`
`gluster peer probe server-4`

4) On server-1, run the following commands to build the GlusterFS Volume:
`gluster volume create HadoopVol  server-1:/mnt/brick1/hadoop server-2:/mnt/brick1/hadoop server-3:/mnt/brick1/hadoop server-4:/mnt/brick1/hadoop `
`gluster volume start HadoopVol`
`gluster volume status`

Mount the Volume
`mkdir /mnt/glusterfs`
`chmod -R 1777 /mnt/glusterfs`
`mount -t glusterfs <host>:/HadoopVol /mnt/glusterfs`

Set permissions for the getfattr process
`vi /etc/sudoers.d/gluster`
`mapred ALL= NOPASSWD: /usr/bin/getfattr`
`chmod 440 /etc/sudoers.d/gluster`