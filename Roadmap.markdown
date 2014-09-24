# Roadmap for glusterfs-hadoop 
##(September 24, 2014)
These are future (v2.2) possible features for glusterfs-hadoop:

**Support for distributed volumes**
1. Support striped volumes (changed parsing of getfattr)

**ACL**
1. Iron out any descrepencies between HDFS ACL and Posix ACL

**Enhancements to FS statistics tracking**
1. read/write, file sizes etc
	
**Quota**
1. pre-checka and warn if low on space
2. misc bugs around quota-pre check

**'rack' locality config support**
1. Allow users to supply a list of hosts to rack mappings
2. Report rack locality in getFileLBlockLocations(..) calls

**libgfapi**
1. Support curret glusterfs-hadoop feature set (volumes, multiple users etc..)
2. No more FUSE config steps
3. separate plugin
4. more control over buffers
5. low expected performance for feature  