## Architecture and Design ##

The plugin currently wraps RawLocalFileSystem implementations for hadoop 1.0 and 2.0.  See the [[Architecture]] page for details.  Once you grok this, there are various ways that you can contribute !

## Some simple (and complex) ways to contribute ##

Some simple ways to contribute: 

- Sharing Examples on this Wiki

- Documentation in the README

- Increasing the usefullness of existing Java Docs

Some more interesting experiments which we haven't yet gotten around to :

- Experimenting with new approaches to opening and optimizing I/O into the Gluster's native file system.

- Experimentation with overlaying aspects to create an easily debuggable file system which can be turned on easily. 

- Using some of Gluster's particularly unique features (near constant time file look ups, highly customizable write paths) to write new types of MapReduce jobs (i.e. jobs which use the DFS as distribute cache for libraries, for example - or jobs which utilize files for interprocess communication).

- Integrating some of our pipeline with the recently released java API's for gluster i/o : https://forge.gluster.org/glusterfs-java-filesystem .  This is an extremely exciting new prospect which may eventually bring java into the gluster universe as a first calss citizen... And it is very relevant to the hadoop plugin for obvious reasons!

Again, before you get started, take a few minutes to look at the existing [[Architecture]] and familiarize yourself with hadoop's highly pluggable FileSystem API. 

## Development Process ##

Development of the plugin is managed on our [github repository](https://github.com/gluster/hadoop-glusterfs). The gluster forge repository is a read-only mirror of the github repository. We enthusiastically welcome any pull requests on our github repository. 

Prior to submitting a pull request, please ensure [that you are able to build the plugin](https://forge.gluster.org/hadoop/pages/HowToBuild) with your changes and that all the build tests pass. If you have additional questions, please use the gluster mailing list that is listed on the project page.
