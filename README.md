
# NiFi Site-to-Site Direct streaming to Storm for Log Ingestion


## Short Description

Sample Application for Log Ingestion with NiFi and Storm into Phoenix using NiFi Site-to-Site on HW Sandbox.

## Introduction

Using NiFi, data can be exposed in such a way that a receiver can pull from it by adding an Output Port to the root process group. 
For Storm, we will use this same mechanism - we will use the Site-to-Site protocol to pull data from NiFi's Output Ports. In this tutorial we learn to capture NiFi app log from the Sandbox and parse it using Java regex and ingest it to Phoenix via Storm or Directly using NiFi PutSql Processor

## Prerequisite

1) Assuming you already have latest version of NiFi-1.x/HDF-2.x downloaded on your HW Sandbox Version 2.5, else execute below after ssh connectivity to sandbox is established:

```
# cd /opt/
# wget http://public-repo-1.hortonworks.com.s3.amazonaws.com/HDF/centos6/2.x/updates/2.0.1.0/HDF-2.0.1.0-centos6-tars-tarball.tar.gz
# tar -xvf HDF-2.0.1.0-12.tar.gz
```
2) Storm is Installed on your VM and started.
3) Hbase is Installed with phoeix Query Server.
4) Make sure Hbase is up and running and out of maintenance mode, below properties are set(if not set it and restart the services):
	- Enable Phoenix --> Enabled
	- Enable Authorization --> Off

## Steps

1) Open **nifi.properties** for updating configurations:

```
# vi /opt/HDF-1.2.0.0/nifi/conf/nifi.properties
```

2) Change NIFI http port to run on 8099 as default 8080 will conflict with Ambari web UI

```
# web properties #
nifi.web.http.port=8099
```

3) Configure NiFi instance to run site-to site by changing below configuration : add a port say 8055 and set "nifi.remote.input.secure" as "false"

```
# Site to Site properties #
nifi.remote.input.socket.host=sandbox
nifi.remote.input.socket.port=8066
nifi.remote.input.secure=false
```

4) Now Start [Restart if already running as configuration change to take effect] NiFi on your Sandbox.

```
# /opt/HDF-1.2.0.0/nifi/bin/nifi.sh start
```

6) Let us build a small flow on NiFi canvas to read app log generated by NiFi itself to feed to Storm:
	
* Connect to below url in your browser: http://your-vm-ip:8099/nifi/
* Drop a  "**TailFile**" Processor to canvas to read lines added to "**/opt/HDF-1.2.0.0/nifi/logs/nifi-app.log**". Auto Terminate relationship Failure. 
* Drop an OutputPort to the canvas and Name it "**OUT**", Once added, connect "TailFile" to the port for Success relationship.

7) Now Lets setup the storm topology, the sample code is available as "**NiFiStormTopology.java**" and "**Storm_Nifi.jar**" contains all dependencies required for the topology to run including NiFi Site-to-Site client.

```
# cd /opt/
# git clone https://github.com/jobinthompu/Storm-NiFi.git
```

8) Lets Go back to the NiFi Web UI and start the flow we created, make sure nothing is wrong and you shall see data flowing and stuck in the connection to output port "**OUT**"

9) Lets submit storm topology with below command [Assuming nifi is running on localhost:8099 and data is available on port 'OUT'. If you need to make any minor changes in the code  you can follow instruction starting step 11. **MyFile_** is the prefix of the local files to be created]

```
# storm jar /opt/Storm-NiFi/resources/Storm_Nifi.jar NiFi.NiFiStormTopology /tmp/MyFile_
```

10) Once topology is submitted, you may check and verify the files received placed under /tmp/ [as storm user is running the topology, makesure storm user have permission to write in the path you provided.]

```
# ls -l /tmp | grep MyFile_
```

## Making Config changes to Topology:

11) To make changes in NiFiStormTopology.java and re-submit the topology, use following steps:

```
# vi /opt/Storm-NiFi/resources/NiFi/NiFiStormTopology.java
```
12) Once changes are done, compile NiFiStormTopology.java file and update the jar with latest classfiles:

```
# cd /opt/Storm-NiFi/resources
# javac -cp ".:Storm_Nifi.jar" NiFi/NiFiStormTopology.java
# jar uf Storm_Nifi.jar  NiFi/
```

13) No you can re-submit the topologies with updated jars [follow step 9].


### References:

* [ bbende's - nifi-storm](https://github.com/apache/nifi/tree/master/nifi-external/nifi-storm-spout/src/main/java/org/apache/nifi/storm)


Thanks,
Jobin George
