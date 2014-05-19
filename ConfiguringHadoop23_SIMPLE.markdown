## Part 1 : Base setup.

First, we'll set up hadoop on gluster, with no security, using the yarn container executor (which is the default).

**After that** we will add the GlusterContainer Executor in as a quick and simple way to run jobs on CDH5 against glusterfs with multitenancy.

[[General_Configuration_CDH5]]

## Part 2 : Adding simple "linux container" based multitenancy 

At this point, you should be able to run jobs as the "yarn" user, but as no other user.  

* Create a container-executor.cfg file , and write it out to /etc/hadoop/conf.  You can do this in the shell like so, for a user "tom".  You can add other users as well in a comma separated list (i.e. `allowed.system.users=tom,mary,joe` )

    echo "yarn.nodemanager.linux-container-executor.group=hadoop`
    banned.users=yarn
    min.user.id=1000
    allowed.system.users=tom" >> /etc/hadoop/conf/ container-executor.cfg

* Copy the above file to all machines on your cluster. 

* Now, you must make sure there is some mechanism to ensure that ll the **users** and **groups** i the above file have identical UIDs or GIDs.   There are many ways to do this.  (1) you can copy /etc/passwd and /etc/group from your head node to all others OR  (2) Follow the IPA based user setup section of  [[ConfiguringHadoop23_SECURE]] OR (3) Use your companies internal LDAP servers to provision system ids for you.    

* Ensure that the entries in allowed.system.users have UID > 1000.  

* Restart your yarn and nodemanager services.  To do this, you can follow the snippet in the TESTING STARTUP""  section of  [[
General_Configuration_CDH5]]


