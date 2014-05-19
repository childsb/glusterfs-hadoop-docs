# Base CDH5 Setup #

1) First, set up a base hadoop on gluster configuration in CDH5, in insecure mode, without linux containers. 

This is the easiest way to get started, and then we can layer security on to the cluster afterwards.

[[General_Configuration_CDH5 Instructions for base setup of CDH 5]]

# Securing your cluster with Kerberos and IPA # 

2) Now, the head node of your cluster, `yum install ipa-server.`

3) Run ipa-client-install on each client.  Enter in the master name as the KDC server when doing this. 

4) Add service principals on the head node using ipa-service-add for resource manager and node manager.

5) Add the hadoop group, the yarn group - using ipa group-add.

6) Add members using "ipa group-add-member" , for each user who will be running hadoop jobs on your cluster.  

Assuming your principals are "rm" / "nm"

7) Now, get keytabs for each of the yarn and nodemanager services, and write them to a temp file.  We'll move it to /etc later.

8) Copy those keytabs to a directory on your local file system for each node.  

9) Configure your hadoop cluster with kerberos key tabs, updating the yarn-site.xml and core-site.xml the standard method.