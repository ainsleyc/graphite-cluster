# graphite-cluster
Dockerized components to run a graphite cluster. Composed of:
* graphite-node: A series of carbon relays and a single graphite api, meant to be run on a single host.
* graphite-relay: A carbon relay node. If running more than one carbon cache, the cluster will require a relay node.
* graphite-api: A graphite api node. If running multiple graphite-node instances, the cluster will require an api node.

### Single Node, Single Carbon Cache
The simplest deployment. Good for testing, or if expected metrics load is small. In this scenario, only a single graphite-node is needed:
```
docker run -d -e CARBON_CONFIG={\"cache_count\":1} -v /opt/graphite/data:/opt/graphite/storage/whisper -p 80:80 -p 2013:2013 -p 2014:2014 ainsleyc/graphite-cluster-node
```
* You'll probably want to bind the docker whisper path to a local path on your host with the '-v' option so that the data won't be lost when the docker is shut down.
* The docker uses the default graphite/carbon ports, which you can bind to the host with the '-p' flag:
  * Carbon line port (2013)
  * Carbon pickle port (2014)
  * Graphite webapp (80)

You can confirm that the node is working by visiting the api page (http://host:80) and checking if carbon/agents data is appearing in the local host path bound with the '-v' option.

### Single Node, Multiple Carbon Caches
For larger write loads you can use multiple carbon caches. A common setup is a single cache for each cpu on the host. To run the node:
```
docker run -d -e CARBON_CONFIG={\"cache_count\":4} -v /opt/graphite/data:/opt/graphite/storage/whisper -p 80:80 -p 2014:2014 -p 2024:2024 -p 2034:2034 -p 2044:2044 ainsleyc/graphite-cluster-node
```
In this setup a relay is needed, so to cache pickle ports need to be exposed with the '-p' option. The pickle ports for multiple caches follow the pattern: [2014, 2024, 2034, ...]

To run the relay:
```
docker run -d -e CARBON_CONFIG={\"cache_hosts\":[{\"host\":\"192.168.0.10\"\,\"ports\":[2014\,2024\,2034\,2044]}]} -p 2003:2003 -p 2004:2004 ainsleyc/graphite-cluster-relay
```
* It's important that the "cache_count" parameter matches between the node and relay.
* The loopback address (127.0.0.1) will not work in the "host" field.
* The ports provided should match the ports exposed in the cache node, **order must match**.
* The docker uses the default carbon ports, which should be exposed with the '-p' option:
  * Carbon line port (2003)
  * Carbon pickle port (2004)

You can confirm that the relay is working by checking if carbon/relays data is appearing in the local host path bound with the '-v' option.

### Multiple Node, Multiple Carbon Caches
TBD
