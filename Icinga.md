# SNMP-based monitoring via Icinga2

## Install and setup

To install / setup I followed and read the [docker-compose-icinga](https://github.com/lippserd/docker-compose-icinga) repo maintained by their CTO. The compose file brings up the following containerised services:
* icinga director
* init-icinga2
  * This is because of a known issue that's mentioned in the compose file:
  ```
  # The Icinga 2 docker image does not support configuration via env vars at the moment.
  # So, we have to ship some configs with this little init container. Referenced in depends_on of the icinga2 service.
  ```
* icinga2 - which depends on both init-icinga2 and the redis db
* icingadb - which depends on mysql and the icingadb-redis
* icingadb-redis
* icingaweb - which depends on mysql
  * There are problems associated with this in the compose file:
    ```
    # Restart Icinga Web container automatically since we have to wait for the database to be ready.
    # Please note that this needs a more sophisticated solution.
    ```
* mysql - which is a mariadb container

And the following volumes:
* icinga2
* icingaweb
* mysql

When put together, we can see the compose stack visually like this:
![icinga-arcitecture](./Diagrams/icinga2-hld.png)


What I _didn't_ know was how the `x-` lines at the top were compose extension fields, used to create re-usable data, ignored by the actual processing of docker-compose, it's ignored untill pulled in later. The use of the `&` and `*` in similar areas just relate that data is being referenced where `&` is the anchor and `*` references it. The final `<<` adds them together. So this compose file section:
```yaml
x-icinga-db-web-config:
  &icinga-db-web-config
  icingaweb.modules.icingadb.config.icingadb.resource: icingadb
  icingaweb.modules.icingadb.redis.redis1.host: icingadb-redis
  ...
```
Is referenced by this line:
```yaml
environment:
      icingaweb.enabledModules: director, icingadb, incubator
      <<: [*icinga-db-web-config, *icinga-director-config, *icinga-web-config]
```
This makes the compose file significantly more streamlined and easier to maintain (if you know what you're doing!)

After copying the compose [playground strucutre](https://github.com/lippserd/docker-compose-icinga) the stack can be brought up via `docker compose up -d` and is accessible at [localhost:8080](http://localhost:8080/authentication/login)

## Director and automation

To get the first host onboarded (my first srx300) it made sense to use director's automation features to define what an SRX300 monitor should look like
