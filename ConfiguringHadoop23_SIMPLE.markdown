#Overview. #

First, we'll set up hadoop on gluster, with no security, using the yarn container executor (which is the default).

Then we will add the GlusterContainer Executor in as a quick and simple way to run jobs on CDH5 against glusterfs with multitenancy.

##Part 1: Setup Hadoop on GlusterFS with CD5.

See  [[General_Configuration_CDH5]] for generic configuration of hadoop on Gluster for CDH5

After this, you can execute jobs, but only as the "yarn" user. 

If you have not already done so, confirm that you can start the yarn services. Export environment variables and then start nodemanager and resourcemanager services.  

_Important!:_ Only start the resource manager on the master node. Omit resourcemanager commands for all slave nodes.

_Note:_ `source env.sh` is only necessary if env.sh is not localted in /etc/profile.d/.  Otherwise, they will be set during on each login to yarn user.

    su yarn
    source env.sh
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start resourcemanager
    /usr/lib/hadoop-yarn/sbin/yarn-daemon.sh start nodemanager

Then check for their java processes as yarn user.

    su yarn -c "jps"

Should return something similar to this (omiting ResourceManager on all slaves):

    <$LVMID>  NodeManager
    <$LVMID>  ResourceManager
    <$LVMID>  Jps


##Part 2: Running Hadoop with the GlusterContainerExecutor

1) Add/modify the following properties in the yarn-site.xml file.

* yarn.nodemanager.container-executor.class = org.apache.hadoop.yarn.server.nodemanager.GlusterContainerExecutor
* yarn.nodemanager.linux-container-executor.group = hadoop

2) Edit /etc/hadoop/conf/container-executor.cfg to mirror the following

    yarn.nodemanager.linux-container-executor.group=hadoop
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom

3) Copy container-executor.cfg to all machines on your cluster.

    scp /etc/hadoop/conf/container-executor.cfg root@$NODE:/etc/hadoop/conf

4) Create user "tom" if it does not already exist on master node.

    useradd -u 1024 -g hadoop tom

5)  All hadoop users must have a UID of or greater than the value of `min.user.id` (1000) in the container-exectuor.cfg.  Any **one** of the following are appropriate ways to ensure identical UID/GID across the cluster.

* 5.1 ) Copy /etc/passwd and /etc/group from your head node to all others  

* 5.2) Follow the IPA based user setup section of  [[ConfiguringHadoop23_SECURE]] OR 

* 5.3) Use your companies internal LDAP servers to provision system ids for you.    


5) Restart your yarn and nodemanager services.  To do this, you can follow the snippet in the [Startup](https://forge.gluster.org/hadoop/pages/General_Configuration_CDH5#Startup)  section of 
General_Configuration_CDH5.

## Part 3: _Finished!_

Test the cluster by executing hadoop jobs as `tom` (or whichever username you chose as job executor).