**WORK IN PROGRESS**

This page outlines how to use our GlusterContainerExecutor to run CDH5 (cloudera hadoop) with simple security, while still maintaining multitenancy.

1) Install the glusterfs-hadoop plugin as you normally would

2) Setup the LinuxContainerExecutor as you normally would.

3) Change the Container executor class in your yarn-site.xml to 

'''
org.apache.hadoop.yarn.server.nodemanager.GlusterContainerExecutor.java
'''

