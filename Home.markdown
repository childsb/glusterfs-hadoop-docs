## Introduction ##

While Apache Hadoop typically ships with the Hadoop Distributed FileSystem (HDFS), [Hadoop can be configured to use alternate FileSystems] (http://wiki.apache.org/hadoop/HCFS) by leveraging its inherent pluggable filesystem architecture. One achieves this by building a plugin that acts as a mediator between the Hadoop FileSystem Interface and the desired FileSystem, and configuring Hadoop to use the plugin. This project contains the plugin required to enable GlusterFS to be used as a Hadoop FileSystem. 

## Releases ##

The Plugin supports both Apache Hadoop 1.x and 2.x releases and has thus far been successfully tested against both the Hortonworks and Intel Hadoop Distributions. The plugin is backwards compatible and thus we recommend you always obtain the latest version (the highest numerical value). [Plugin releases are available on our archiva hosted maven repo](http://23.23.239.119/archiva/browse/org.apache.hadoop.fs.glusterfs/glusterfs-hadoop) 

## Configuration ##

Our [[Configuration]] page provides a guide on how to configure GlusterFS and Hadoop to make use of the plugin.

## How To Contribute ##

Our [[HowToContribute]] page contains information on how to participate in the development of the plugin.

## How To Build ##

Our [[HowToBuild]] page contains information on how to build the plugin if you would like to build your own plugin JARs.  Building the plugin is simple: only maven and java are required. 