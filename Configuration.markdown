## Solution Components ##

The following components are required to successfully deploy a working solution. More detail will be provided on these components later in this guide.

* GlusterFS 3.3 +
* FUSE Kernel Patch
* Apache Hadoop 1.x or 2.x
* The GlusterFS Hadoop FileSystem Plugin
* Oracle Java Runtime Environment (JRE) 1.6 +
* NTP

## Configuration Guide ##

This guide is focused on giving the community a way to quickly evaluate Hadoop on GlusterFS. A popular way to try the solution out is to set up 4 Virtual Machines with Fedora 19 and then follow the instructions below.

** Installing and Configure GlusterFS ** 

Prior to Installing and Configuring Hadoop, once needs to first install GlusterFS across the cluster and configure a Distributed Replicated 2 GlusterFS volume.  This is easy to do [with our automated deployment tool](https://forge.gluster.org/hadoop/pages/GlusterfsClusterInstall). You can also [do this manually](https://forge.gluster.org/hadoop/pages/InstallingAndConfiguringGlusterFS) 

Additional Gluster Documentation and Downloads [are available here](http://www.gluster.org/download/)

** Install Oracle Java 1.6 **

This is a requirement of Hadoop. Hadoop has not yet been widely tested with OpenJDK, although it may well work. The JRE needs to be installed on every server within the trusted storage pool that your Gluster Volume uses. [Download it here](http://www.oracle.com/technetwork/java/javase/downloads/jdk6u38-downloads-1877406.html)

** Configuring Hadoop related tools **

**For Hadoop 1.x:** please see - [Configuring Hadoop 1.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop1) for GlusterFS

**For Hadoop 2.x:** please see - [Configuring Hadoop 2.0](https://forge.gluster.org/hadoop/pages/ConfiguringHadoop2) for GlusterFS

**For Cloudera Hadoop 5 (CDH5): Simple Security Mode ** 
please see - [CDH5 (simple) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SIMPLE) for GlusterFS

**For Cloudera Hadoop 5 (CDH 5): Full security mode ** 

please see - [CDH5 (kerberos) ] (https://forge.gluster.org/hadoop/pages/ConfiguringHadoop23_SECURE) for GlusterFS


**User Guide:** If you have questions about using specific components within the Hadoop Ecosystem, [please see the user guide](https://forge.gluster.org/hadoop/pages/UserGuide)

**Sync Clocks**
		
It is important that the time and date be synchronized for each server within the cluster.   The Network Time Protocol (NTP) is a popular method to achieve this. 
		
To install NTP on each node run:
		
   yum install ntp
		
To update the clock on each machine run:
		     
    ntpd -qg
		
Consult the ntpd documentation to configure periodic time resync.

** Troubleshooting Tips ** 
[Troubleshooting Hadoop Install](https://forge.gluster.org/hadoop/pages/TroubleShootHadoop)