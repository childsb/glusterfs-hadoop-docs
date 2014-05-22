First, Follow the [[General_Configuration_CDH5/]] setup.  

Then, Follow the "Setup Linux Container" Instructions on the Bottom of [[ConfiguringHadoop23_SIMPLE]]

Now, we will setup kerberos:

# STOP all hadoop services and remove hadoop users. # 

* Stop all hadoop services.  You can do this quickly with `killall -9 java` or you can find pids for the NodeManager and ResourceManager processes running  "jps" (do this as root, so you're gauranteed to see all of them), and kill them directly.

* Setup the linux container executor in your yarn-site.xml, add/modify the following properties:

    yarn.nodemanager.container-executor.class=     org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor

    yarn.nodemanager.linux-container-executor.group=
        hadoop

# Set up IPA #

- Now, the head node of your cluster, `yum install ipa-server.`

-  Run `ipa-client-install` on each client.  Enter in the master name as the KDC server when doing this. 

-  Add service principals on the head node using ipa-service-add for resource manager and node manager.

-  Add the hadoop group, the yarn group - using ipa group-add.

-  Add members using "ipa group-add-member" , for each user who will be running hadoop jobs on your cluster.   For example, user tom would be added using ipa.  

Since you need a resourcemanager and nodemanager principal your principals are "rm" / "nm"

-  Now, get keytabs for each of the yarn and nodemanager services, and write them to a temp file.  We'll move it to /etc later.

-  Copy those keytabs to a directory on your local file system for each node.  

-  Configure your hadoop cluster with kerberos key tabs, updating the yarn-site.xml and core-site.xml the standard method.

Note that: In this example, we reused keytab/service name for the first machine on all other nodes of cluster.  That is a bit of a comprimise in terms of security : it means that if someone intercepts the keytab, all machines are comprimised with respect to that particular service.

# Follow Standard Kerberos Setup #

On your core-site.xml:

* set hadoop.security.authentication=true
* set hadoop.security.authorization=true
* Add the following entry: 
    <name>hadoop.security.auth_to_local</name>
    <value>
        RULE:[1:$1@$0](.*@YOUR_REALM)s/@.*//
        DEFAULT
    </value>

On your yarn-site.xml: 

* set yarn.resourcemanager.keytab=/etc/hadoop/conf/rm.keytab
* yarn.nodemanager.keytab=nm/YOUR_HEAD_NODE@YOUR_REALM
* yarn.resourcemanager.keytab=rm/YOUR_HEAD_NODE@YOUR_REALM

# Set user passwords for users # 

You can set sally password through the free ipa web ui : http://www.freeipa.org/page/Web_UI.  

# Log in a user and run a job # 

Now, you will want to run "kinit yarn", and restart all services.  

Then kerberos to login a hadoop user, i.e. `kinit sally`... and enter sally's password. 
