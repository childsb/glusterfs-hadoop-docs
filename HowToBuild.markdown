# How to Contribute #

## Architecture and Design

The plugin currently wraps RawLocalFileSystem implementations for hadoop 1.0 and 2.0.  See the [[Architecture]] page for details.  Once you grok this, there are various ways that you can contribute !

## Some simple (and complex) ways to contribute

Wether you're a Linux guru, a DevOps smarty pants, a JVM ninja, a MapReduce hacker, or maybe just a wiki-formatting expert -- we'd love to get some help from you... And you will learn a lot of cool stuff working on this project. Check out the Hack Ideas below for some ideas.

## Development Process

Development of the plugin is managed on our [github repository](https://github.com/gluster/hadoop-glusterfs). The gluster forge repository is a read-only mirror of the github repository. We enthusiastically welcome any pull requests on our github repository. 

Prior to submitting a pull request, please ensure that you are able to build the plugin (_see below_) with your changes and that all the build tests pass. If you have additional questions, please use the gluster mailing list that is listed on the project page.
------------------------------
# How to Build #

## Pre-Requisites

- You *should* have a working gluster mount for unit tests, although they may feasibly pass without a gluster mount since the proxy writes into a local file system, at least for versions 1.x and 2.x of the plugin.

- [Maven] (supported version 3.1, the build is known to fail on older maven/surefire combinations which seem not to support @Ignore annotations - currently investigating this).  (http://maven.apache.org/).

- JDK  1.6+.  OpenJDK is fine for development.   Different deployments may use different vendor specific JVM's, however, so you should always test your MapReduce code according to whats on your cluster.

## ECLIPSE DEVELOPMENT

1) Setup gluster mount /mnt/glusterfs or whatever. 

    mount -t glusterfs -o acl localhost:/HadoopVol /mnt/glusterfs 
   
2) To run any unit tests, or all unit tests click your project, click on "Run As", click "Run Configurations", and add these parameters as "VM Arguments"

-DHCFS_FILE_SYSTEM_CONNECTOR=org.apache.hadoop.fs.test.connector.glusterfs.GlusterFileSystemTestConnector 

-DHCFS_CLASSNAME=org.apache.hadoop.fs.glusterfs.GlusterFileSystem 

-DHCFS_CLASSNAME=org.apache.hadoop.fs.glusterfs.GlusterFileSystem 

-DGLUSTER_MOUNT=/mnt/glusterfs

## Building with MAVEN

export HCFS_FILE_SYSTEM_CONNECTOR="org.apache.hadoop.fs.test.connector.glusterfs.GlusterFileSystemTestConnector" 
export DHCFS_CLASSNAME="org.apache.hadoop.fs.glusterfs.GlusterFileSystem" 
export HCFS_CLASSNAME="org.apache.hadoop.fs.glusterfs.GlusterFileSystem" 
export GLUSTER_MOUNT="/mnt/glusterfs"

And then run: 

mvn package; 

You will see the shim written out to the target/ folder.
---------------------------------
# Hack Ideas #

1) Some simple ways to contribute: 

- Sharing Examples on this Wiki

- Documentation in the README

- Increasing the usefullness of existing Java Docs

2) Some more interesting experiments which we haven't yet gotten around to if you are a coming from the gluster world.

- Experimenting with new approaches to opening and optimizing I/O into the Gluster's native file system.

- Updating and creating tests for a more robust java API for accessing XATTRs.

3) If you're a Linux or DevOps guru, we'd love to have some feedback and contributions in these areas :

- Building developer VMs (i.e with KVM, Vagrant) which can be automatically spun up and used for direct integration with our CI.  A great place to start with this is the bigtop project, which creates hadoop centric VMs for the apache hadoop community.  Such VMs, if created for our purposes, might be useful to the broader gluster community also.

- Help writing automated, reproduceable tests that might expose gotcha's with distributed access of FUSE mounted distributed file systems, privileges, etc.. 

- Curating and evaluating puppetization resources for gluster on top of hadoop.  

4) And finally, if you are a MapReduce hacker, here are some other ways you can get involved.

- Experimentation with overlaying aspects to create an easily debuggable file system which can be turned on easily. 

- Using some of Gluster's particularly unique features (near constant time file look ups, highly customizable write paths) to write new types of MapReduce jobs (i.e. jobs which use the DFS as distribute cache for libraries, for example - or jobs which utilize files for interprocess communication).

- Integrating some of our pipeline with the recently released java API's for gluster i/o : https://forge.gluster.org/glusterfs-java-filesystem .  This is an extremely exciting new prospect which may eventually bring java into the gluster universe as a first calss citizen... And it is very relevant to the hadoop plugin for obvious reasons!

- Scouring the RawLocalFileSystem and other base classes for optimizations of and/or other hadoop FileSystem workloads.  

Again, before you get started, take a few minutes to look at the existing [[Architecture]] and familiarize yourself with hadoop's highly pluggable FileSystem API. 
