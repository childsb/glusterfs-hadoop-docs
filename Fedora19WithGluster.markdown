yum install glusterfs glusterfs-server glusterfs-fuse
service  glusterd start
(this will fail, but its required to get glusterd into the /usr/sbin directory)

# On each server, do the following:
cd /usr/sbin
./glusterd

# Stop the Firewall so you can successfully peer probe
service iptables stop
chkconfig iptables off
systemctl stop firewalld.service

# Create your brick. This can a single directory or a block device you mount onto this directory
mkdir /mnt/brick1

# On server-1, Peer Probe to create a trusted storage pool 
gluster peer probe server-2

# Create the Gluster Volume
gluster volume create HadoopVol  <host>:/mnt/brick1 int3.rhs:/mnt/brick1  <host>:/mnt/brick1
gluster volume start HadoopVol
gluster volume status

# Mount the Volume
mkdir /mnt/glusterfs
chmod -R 1777 /mnt/glusterfs
mount -t glusterfs <host>:/HadoopVol /mnt/glusterfs

# Set permissions for the getfattr process
vi /etc/sudoers.d/gluster
mapred ALL= NOPASSWD: /usr/bin/getfattr
chmod 440 /etc/sudoers.d/gluster