# Observability Lab

To gain a better understanding of how modern-day network observability works I've built this repo to cover all areas of network observability:
* (Traffic) stats monitoring
* Hardware monitoring
* Protocol monitoring
* Logging ingestion

To do this I'm using this repo as a platform environment to further push my understanding of 'as-code' princples while running the monitoring applications as containers, managed in the `docker-compose.yaml` in the root of the directory.

These containerised applications include:
* Icinga for the core monitoring
  * Also featuring icinga-director (for configuration managment)
* Grafana for visualisation

For hardware, I'll be monitoring my Juniper EX-2200-C switches and two SRX-300 firewalls using SNMP first.

## SNMP-based monitoring via Icinga2

To install / setup I followed and read the [docker-compose-icinga](https://github.com/lippserd/docker-compose-icinga) repo maintained by their CTO.
