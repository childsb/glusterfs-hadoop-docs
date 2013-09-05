

Concretely, one can look at any of the FileSystem or DelegateToFileSystem implementations and note that (most) functionality is passed to a single class: the GlusterVolume, which is wrapped in different ways to satisfy the requirements of different top level file systems.  Actually, the GlusterVolume class itself is also a file system: It implements the RawLocalFileSystem class.  The classes that get used by hadoop, however, need a little extra wrapping, and thus we provide the GlusterVolume as a utility FileSystem which can be wrapped by other FileSystem implementations with fine grained behaviour.  

As an example, this can be seen by comparing the current GlusterFileSystem with the GlusterFileSystemCRC implementation: Both are quite similar, with exception of the fact the GlusterFileSystem extends a class which is "higher" in the FileSystem inheritance hierarchy. 

The hadoop plugin provides FileSystem and AbstractFileSystem implementations for hadoop 1.0 and 2.0, respectively.  

* The hadoop 1.0 Gluster implementations of FilterFileSystem and LocalFileSystem (i.e. CRC File System) classes are implemented in the org.apache.hadoop.fs.glusterfs package, where the GlusterVolume is a backing class that adds gluster semantics to a wrapper to the RawLocalFileSystem.  
In this case, both FileSystem implementations send a GlusterVolume as constructor argument to proxy their respective implementations.

* The hadoop 2.0 follows the same essential architecuture, ultimately delegating down to the GlusterVolume class.  Instead, however, the hadoop 2.0 classes implement a DelegateToFileSystem proxy.  Which calls an underlying GlusterVol (2.0 wrapper to the GlusterVolume class) for its implementation.  

Informally, we have 5 "file system" implementations:  The GlusterFileSystem, the GlusterFileSystemCRC, the GlusterFs, and the GlusterFsCRC.  The former of these are for hadoop 1.0, and the latter of these are hadoop 2.0 (i.e. Abstract File System) classes.