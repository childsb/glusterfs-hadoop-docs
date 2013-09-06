
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
