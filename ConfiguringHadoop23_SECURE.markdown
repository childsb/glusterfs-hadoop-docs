Follow the [[ConfiguringHadoop23_SIMPLE]] setup.  

# STOP all hadoop services and remove hadoop users. # 

* Stop all hadoop services.  You can do this quickly with `killall -9 java` or you can find pids for the NodeManager and ResourceManager processes running  "jps" (do this as root, so you're gauranteed to see all of them), and kill them directly.

Now, we will change the GlusterContainerExecutor to the LinuxContainerExecutor.  Afterwards, we will move over to using kerberos authentication for the hadoop group, the yarn user, and our other system users.

* Setup the linux container executor in your yarn-site.xml, add/modify the following properties:

    yarn.nodemanager.container-executor.class=     org.apache.hadoop.yarn.server.nodemanager.LinuxContainerExecutor

    yarn.nodemanager.linux-container-executor.group=
        hadoop

# Set up IPA #

1) As root, remove or back up any data for users deleted in the previous step.  You can restore that data later.  

2) Now, the head node of your cluster, `yum install ipa-server.`

3) Run `ipa-client-install` on each client.  Enter in the master name as the KDC server when doing this. 

4) Add service principals on the head node using ipa-service-add for resource manager and node manager.

5) Add the hadoop group, the yarn group - using ipa group-add.

6) Add members using "ipa group-add-member" , for each user who will be running hadoop jobs on your cluster.   For example, user tom would be added using ipa.  

Assuming your principals are "rm" / "nm"

7) Now, get keytabs for each of the yarn and nodemanager services, and write them to a temp file.  We'll move it to /etc later.

8) Copy those keytabs to a directory on your local file system for each node.  

9) Configure your hadoop cluster with kerberos key tabs, updating the yarn-site.xml and core-site.xml the standard method.