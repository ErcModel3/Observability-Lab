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

These can be done with the following config:
```
--{ * candidate shared default }--[  ]--
A:admin@edge1-pen# diff
      system {
          grpc-server mgmt {
+             metadata-authentication true
+             tls-profile tls-profile-1
+             port 50052
          }
      }

--{ * candidate shared default }--[  ]--
A:admin@edge1-pen#
```
