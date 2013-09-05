Overview

The major functionality of the hadoop plugin is provided via the GlusterVolume class, which is a wrapper to the underlying FUSE file system.  The GlusterVolume class can be wrapped as needed into classes which use existing frameworks in the hadoop FileSystem class hierarchy which implement different behaviours.  Unlike the Hadoop DistributedFileSystem, which is contains complex logic for managing metadata and write paths, the hadoop plugin relies on an underlying gluster installation which can be accessed through a local FUSE mount.  Thus, the architecture of the plugin is to use the local file system implementations (RawLocalFileSystem) in hadoop which are commonly used by developers who are writing and testing local MapReduce code.    

Ultimately, the major "interesting" operations that are done when interacting with gluster are done through a FUSE abstraction, and thus, the code for the hadoop plugin is dominated mostly by hooks which allow one to write to a glusterfs:// URI from the various hadoop interfaces.   In later implementations, this might be modified (i.e. more significant gluster functionality will be customized on top of, or independently of, the underlying FUSE layer).

(tl;dr version)  

The hadoop plugin for gluster is mainly an abstraction on top of the RawLocalFileSystem implementations which are provided for us by the hadoop community. 

The Details

It is easiest to grok the architecture by scanning the class hierarchy in eclipse or another IDE.  To start, you can look at any of the FileSystem or DelegateToFileSystem implementations and note that (most) functionality is passed to a single class: the GlusterVolume, which is wrapped in different ways to satisfy the requirements of different top level file systems.  Actually, the GlusterVolume class itself is also a file system: It implements the RawLocalFileSystem class.  The classes that get used by hadoop, however, need a little extra wrapping, and thus we provide the GlusterVolume as a utility FileSystem which can be wrapped by other FileSystem implementations with fine grained behaviour.  

As an example, this can be seen by comparing the current GlusterFileSystem with the GlusterFileSystemCRC implementation: Both are quite similar, with exception of the fact the GlusterFileSystem extends a class which is "higher" in the FileSystem inheritance hierarchy. 

The hadoop plugin provides FileSystem and AbstractFileSystem implementations for hadoop 1.0 and 2.0, respectively.  

* The hadoop 1.0 Gluster implementations of FilterFileSystem and LocalFileSystem (i.e. CRC File System) classes are implemented in the org.apache.hadoop.fs.glusterfs package, where the GlusterVolume is a backing class that adds gluster semantics to a wrapper to the RawLocalFileSystem.  
In this case, both FileSystem implementations send a GlusterVolume as constructor argument to proxy their respective implementations.

* The hadoop 2.0 follows the same essential architecuture, ultimately delegating down to the GlusterVolume class.  Instead, however, the hadoop 2.0 classes implement a DelegateToFileSystem proxy.  Which calls an underlying GlusterVol (2.0 wrapper to the GlusterVolume class) for its implementation.  

Informally, we have 5 "file system" implementations:  The GlusterFileSystem, the GlusterFileSystemCRC, the GlusterFs, and the GlusterFsCRC.  The former of these are for hadoop 1.0, and the latter of these are hadoop 2.0 (i.e. Abstract File System) classes.