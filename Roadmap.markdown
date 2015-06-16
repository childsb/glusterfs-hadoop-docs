# Roadmap for glusterfs-hadoop 
##(September 24, 2014)
These are future (v2.2) possible features for glusterfs-hadoop:

**Support for striped volumes**
1. Support striped volumes (requires changed parsing of getfattr)

**ACL**
1. Iron out any descrepencies between Hadoop ACL and Posix ACL

** Non-Hadoop Apps **
1. Support for SPARK and Tachyon using the same Hadoop FileSystem plugin.

**Enhancements to FS statistics tracking**
1. Better tracking of read/write, file sizes etc
	
**Quota**
1. Pre-check quota and warn if low on space
2. Fix misc bugs around quota-pre check

**'Rack' locality config support**
1. Allow users to supply a list of hosts to rack mappings
2. Report rack locality in getFileLBlockLocations(..) calls

**libgfapi**
1. Support curret glusterfs-hadoop feature set (volumes, multiple users etc..)
2. Eliminate OS FUSE config steps
3. Allows more control over buffering
4. Incremental performance improvement  