# Observability Lab

To gain a better understanding of how modern-day network observability works I've built this repo to cover all areas of network observability:
* (Traffic) stats monitoring
* Hardware monitoring
* Protocol monitoring
* Logging ingestion

To do this I'm using containerlab running a single SR Linux (IXR-D5 on v25.10.1) node with prometheus as per the following diagram:

![](/Diagrams/hld.png)

All the lab is (at this stage) is a single SR Linux node called edge1-pen with a Prometheus node that makes HTTP queries to the switch.

## Switch setup and config

To start I need to actually confirm how the switch exports stats and if it has a default nethod for exporting stats. This can be found on Nokia's [SR Linux 25.10 documentation](https://documentation.nokia.com/srlinux/25-10/index.html) and then the [SR Linux Prometheus Exporter](https://learn.srlinux.dev/ndk/apps/srl-prom-exporter/#sr-linux-prometheus-exporter) which is a prometheus scrape-able endpoint that can run on a switch.

The Prometheus exporter is an [SR Linux NDK](https://learn.srlinux.dev/ndk/#netops-development-kit-ndk) (netops development kit) agent that exposes a Prometheus endpoint ready for scraping. It uses SR Linux's unix socket address to retrieve gNMI (gRPC Network Management Interface) updates. With those updates it bulds the Prometheus metrics and exposes them via an HTTP server.

For the exporter to work the following need to be configured:
* The gNMI server unix socket is enabled
* The HTTP address and port are accepted by the SRL CPM filters

To start the [gRPC Server](https://documentation.nokia.com/srlinux/25-10/books/config-basics/management-servers.html#ariaid-title3) (an open source Remote Procedure Call framework) in SR linux

The gRPC server  can be done with the following config:
```
set / system grpc-server mgmt admin-state enable
set / system grpc-server mgmt rate-limit 65000
set / system grpc-server mgmt network-instance mgmt
set / system grpc-server mgmt port 50052
set / system grpc-server mgmt trace-options [ request response common ]
set / system grpc-server mgmt services [ gnmi ]
set / system grpc-server mgmt source-address [ 10.0.67.1 ]
set / system grpc-server mgmt unix-socket admin-state enable
```
And the HTTP server and port can be defined with this:
```
insert / acl acl-filter cpm type ipv4 entry 500 match ipv4 protocol tcp
insert / acl acl-filter cpm type ipv4 entry 500 description "Accept HTTP traffic on 8888"
insert / acl acl-filter cpm type ipv4 entry 500 match transport destination-port value 8888
```
And can be validated with:
```
--{ running }--[  ]--
A:root@edge1-pen# show acl acl-filter cpm type ipv4 entry 500
==================================================================================================================================================================================================================
Filter        : cpm
SubIf-Specific: disabled
Entry-stats   : yes
Entries       : 1
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Entry 500
  Match               : protocol=tcp, (*)->(8888-8888)
  Action              : none
  Match Packets       : 0
  Last Match          : never
  TCAM Entries        : 1 for one subinterface and direction
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

--{ running }--[  ]--
A:root@edge1-pen#
```

## Install
To insall the Exporter NDK agent, you need to:
1. Download the [RPM](https://github.com/karimra/srl-prometheus-exporter/releases) from the releases tab and copy it to the switch using docker cp.
```
docker cp srl-prometheus-exporter_0.2.14_Linux_x86_64.rpm clab-Lab-edge1-pen:/tmp

Successfully copied 5.89MB to clab-Lab-edge1-pen:/tmp
```
2. Dropping into the shell and confirming the install
```
# Entering the shell and validating the install
--{ + running }--[  ]--
A:admin@edge1-pen# bash
admin@edge1-pen:~$ 
 
admin@edge1-pen:~$ ls /tmp
default_gateway.txt  snmp_debug  srl-prometheus-exporter_0.2.14_Linux_x86_64.rpm  topology.yml
```
3. Installing the RPM agent module
```

```