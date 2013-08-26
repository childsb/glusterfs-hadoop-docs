## Development Process ##

Development of the plugin is managed on our [github repository](https://github.com/gluster/hadoop-glusterfs). The gluster forge repository is a read-only mirror of the github repository. We enthusiastically welcome any pull requests on our github repository. 

Prior to submitting a pull request, please ensure [that you are able to build](https://forge.gluster.org/hadoop/pages/HowToBuild) the plugin with your changes and that all the build tests pass.

If you have additional questions, please use the gluster mailing list that is listed on the project page.

## Overview of Libraries ##

* Source Layout (./src/)

org.apache.hadoop.fs.glusters/GlusterFSBrickClass.java
org.apache.hadoop.fs.glusters/GlusterFSXattr.java            <--- Fetch/Parse Extended Attributes of a file
org.apache.hadoop.fs.glusters/GlusterFUSEInputStream.java    <--- Input Stream (instantiated during open() calls; quick read from backed FS)
org.apache.hadoop.fs.glusters/GlusterFSBrickRepl.java
org.apache.hadoop.fs.glusters/GlusterFUSEOutputStream.java   <--- Output Stream (instantiated during creat() calls)
org.apache.hadoop.fs.glusters/GlusterFileSystem.java         <--- Entry Point for the plugin (extends Hadoop FileSystem class)

org.gluster.test.AppTest.java                  <--- Your test cases go here (if any :-))

./tools/build-deploy-jar.py                                                  <--- Build and Deployment Script
./conf/core-site.xml                                                         <--- Sample configuration file
./pom.xml                                                                    <--- build XML file (used by maven)

./COPYING                                                                    <--- License
./README                                                                     <--- This file
