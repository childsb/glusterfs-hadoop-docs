## Just curious ? ## 

If you are just testing the plugin and want to experiment with Hbase, we have a vagrant-gluster-hbase on fedora19 setup which can be run to automatically provision a 2 node gluster and hbase cluster with smoke tests: 

git clone https://forge.gluster.org/vagrant/fedora19-gluster/ 
cd gluster-hbase-example
vagrant up

^^ yes its that easy :)

## Introduction to HBase ## 

HBase setup is complex - you need a zookeeper service to elect and maintain masters, region servers to store your and shard your data, and an HMaster to coordinate the table information and monitor region servers. 

So , first make sure you understand how hbase works before following this tutorial:

http://hbase.apache.org/book/master.html

http://hbase.apache.org/book/zookeeper.html

http://hbase.apache.org/book/regionserver.arch.html

## HBase basic setup instructions ##

Now: to set up, we assume that java , glusterfs, attr, and psmisc are installed, and that there is a gluster volume mounted on /mnt/glusterfs, and that each region server in your cluster has that volume mounted:

- wget hbase from your preferred apache mirror as a tarball

- setup passwordless ssh from your master node to intended region server nodes 

- disable security by turning off iptables, firewalld.service , and setenforce 0

- untar hbase on each node 

- wget  /mnt/glusterfs/lib/

- symlink to /mnt/glusterfs/lib from inside of hbase/conf/lib

- on each node, update your hbase-env.sh script to point to your java installation.

- run the following smoke test to confirm operation

(more to come) 