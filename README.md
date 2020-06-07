Envoy Zone Routing Demo
=======================

This is some example code to accompany a blog post on the zone-aware routing abilities of Envoy. Each subdirectory is a different sandbox scenario. Simply pick one and run `docker-compose`.

## Running
The sanboxes require:
 - curl
 - docker-compose

```bash
$ git clone git@github.com:derekargueta/envoy-zone-routing-demo.git
$ cd envoy-zone-routing-demo/direct_routing
$ docker-compose up -d
$ while true; do curl -so /dev/null localhost:8000; done
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 0
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 7694
cluster.server.lb_zone_routing_cross_zone: 0
cluster.server.lb_zone_routing_sampled: 0
```

## Sandbox Descriptions

### direct_routing
the percentage of zone A clients 2/3 clients and 2/3 servers, so any traffic the zone
A client receives will stay in zone A.

### disabled
No zone aware routing. This is accomplished by removing `min_cluster_size` which defaults to 6. `min_cluster_size` is the minimum number of endpoints a cluster needs for zone-aware routing to be considered, which the sandbox only has 3. Kind of a pointless way to do it, but it only involved removing 3 lines of config and was where my cursor was closest to `¯\_(ツ)_/¯`.

```bash
$ while true; do curl -so /dev/null localhost:8000; done
^C%
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 14
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 0
cluster.server.lb_zone_routing_cross_zone: 0
cluster.server.lb_zone_routing_sampled: 0
```

### cross_zone
2/3 traffic stays in zone A, 1/3 spills over to zone B. This done by configuring as

| zone | clients | servers |
|------|---------|---------|
| a    | 2       | 1       |
| b    | 1       | 2       |

```bash
$ while true; do curl -so /dev/null localhost:8000; done
^C%
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 0
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 0
cluster.server.lb_zone_routing_cross_zone: 85
cluster.server.lb_zone_routing_sampled: 173
```
