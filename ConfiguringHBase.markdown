## Vagrant setup for 2 node test cluster ##

If you are just testing the plugin and want to experiment with Hbase, we have a vagrant-gluster-hbase on fedora19 setup which can be run to automatically provision a 2 node gluster and hbase cluster with smoke tests: 

git clone https://forge.gluster.org/vagrant/fedora19-gluster/ 
cd gluster-hbase-example
vagrant up

## Detailed Instructions (WIP) ##

For HBase to be running, you really will need to enable two services: Zookeeper and HBase.  Zookeeper can be started for you by HBase, if you configure it to do so.  In any case, you need a zookeeper service to elect and maintain masters, region servers to store your and shard your data, and an HMaster to coordinate the table information and monitor region servers.  Most important of all is to note that your /etc/hosts has to be perfect for everything to work properly, based on the fragile nature of how HBase connects to hosts. 

So , first make sure you understand how hbase works before following this tutorial:

http://hbase.apache.org/book/master.html

http://hbase.apache.org/book/zookeeper.html

http://hbase.apache.org/book/regionserver.arch.html


Now: to set up, we assume that java , glusterfs, attr, and psmisc are installed, and that there is a gluster volume mounted on /mnt/glusterfs, and that each region server in your cluster has that volume mounted:

- wget hbase from your preferred apache mirror as a tarball

- setup passwordless ssh from your master node to intended region server nodes 

- disable security by turning off iptables, firewalld.service , and setenforce 0

- untar hbase on each node 

- wget  -O /mnt/glusterfs/lib/glusterfs-hadoop.jar http://23.23.239.119/archiva/repository/snapshots/rhbd/glusterfs-hadoop/2.1.4/glusterfs-hadoop-2.1.4.jar 

- On each node, symlink to /mnt/glusterfs/lib from inside of hbase/conf/lib

ln -s  /mnt/glusterfs/lib/glusterfs-hadoop.jar /home/vagrant/hbase-0.94.11/lib/glusterfs-hadoop.jar

- on each node, update your hbase-env.sh script to point to your java installation:

export JAVA_HOME="/usr/lib/jvm/java-1.7.0-openjdk.x86_64"

## Setting configuration properties ##

Set the following properties: 

```
hbase.zookeeper.property.clientPort = (zookeeper properties file)
hbase.cluster.distributed = true
hbase.zookeeper.quorum = (select 1,3,5... zk machines)
hbase.zookeeper.property.dataDir = (data dir property in /usr/lib/zookeeper...)

```

## Startup ##

You can start up a region server and an hbase master easily like this 

```
./bin/hbase-daemon.sh start zookeeper 
./bin/hbase-daemon.sh start master 
./bin/hbase-daemon.sh start regionserver 
``


## Testing ## 

- run the following smoke test to confirm operation

        sudo hbase-0.94.11/bin/hbase shell -d <<EOF
create 't1','f1' 
put 't1', 'row1', 'f1:a', 'val1'
scan 't1'
EOF
 

## Troubleshooting ##

1) If you get zookeeper or HMaster is down exceptions, make sure your /etc/hosts file should look something like this: 

127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
99.10.10.11	rs1
99.10.10.12	hmaster

(note that in the above, we have localhost = 127.0.0.1, and that there is a mapping both for 127.* and for the external IP 99.10.10.11).  
 