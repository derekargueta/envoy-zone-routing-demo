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
```

## Sandbox Descriptions

### direct_routing
the percentage of zone A clients 2/3 clients and 2/3 servers, so any traffic the zone
A client receives will stay in zone A.

| zone | clients | servers |
|------|---------|---------|
| a    | 2       | 2       |
| b    | 1       | 1       |

```bash
$ cd direct_routing
$ docker-compose up -d
$ while true; do curl -so /dev/null localhost:8000; done
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 0
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 711
cluster.server.lb_zone_routing_cross_zone: 0
cluster.server.lb_zone_routing_sampled: 0
$ curl -s localhost:9901/stats | grep server | grep "server.zone" | grep -v "time"
cluster.server.zone.zone_a.zone_a.upstream_rq_200: 711
cluster.server.zone.zone_a.zone_a.upstream_rq_2xx: 711
cluster.server.zone.zone_a.zone_a.upstream_rq_completed: 711
$ docker-compose stop
```
Zone A keeps 100% of it's traffic in zone A. This is the "perfect" scenario, where the ratio of client:server across all zones is 1:1, but this is rarely true. If we added a zone B front proxy we could also demonstrate 100% of zone B traffic staying in zone B.

### disabled
No zone aware routing AKA regular round-robin. This is accomplished by removing `min_cluster_size` which defaults to 6. `min_cluster_size` is the minimum number of endpoints a cluster needs for zone-aware routing to be considered, which the sandbox only has 3. Kind of a pointless way to do it, but it only involved removing 3 lines of config and was where my cursor was closest to `¯\_(ツ)_/¯`.

| zone | clients | servers |
|------|---------|---------|
| a    | 2       | 2       |
| b    | 2       | 2       |

```bash
$ cd disabled
$ docker-compose up -d
$ while true; do curl -so /dev/null localhost:8000; done
^C%
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 21
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 0
cluster.server.lb_zone_routing_cross_zone: 0
cluster.server.lb_zone_routing_sampled: 0
$ curl -s localhost:9901/stats | grep server | grep "server.zone" | grep -v "time"
cluster.server.zone.zone_a.zone_a.upstream_rq_200: 318
cluster.server.zone.zone_a.zone_a.upstream_rq_2xx: 318
cluster.server.zone.zone_a.zone_a.upstream_rq_completed: 318
cluster.server.zone.zone_a.zone_b.upstream_rq_200: 316
cluster.server.zone.zone_a.zone_b.upstream_rq_2xx: 316
cluster.server.zone.zone_a.zone_b.upstream_rq_completed: 316
$ docker-compose stop
```
Zone A sends 50% of it's traffic to zone B.

### cross_zone
2/3 traffic stays in zone A, 1/3 spills over to zone B.

| zone | clients | servers |
|------|---------|---------|
| a    | 2       | 1       |
| b    | 1       | 2       |

```bash
$ cd cross_zone
$ docker-compose up -d
$ while true; do curl -so /dev/null localhost:8000; done
^C%
$ curl -s localhost:9901/stats | grep server | grep lb_zone
cluster.server.lb_zone_cluster_too_small: 0
cluster.server.lb_zone_no_capacity_left: 0
cluster.server.lb_zone_number_differs: 0
cluster.server.lb_zone_routing_all_directly: 0
cluster.server.lb_zone_routing_cross_zone: 173
cluster.server.lb_zone_routing_sampled: 346
$ curl -s localhost:9901/stats | grep server | grep "server.zone" | grep -v "time"
cluster.server.zone.zone_a.zone_a.upstream_rq_200: 346
cluster.server.zone.zone_a.zone_a.upstream_rq_2xx: 346
cluster.server.zone.zone_a.zone_a.upstream_rq_completed: 346
cluster.server.zone.zone_a.zone_b.upstream_rq_200: 173
cluster.server.zone.zone_a.zone_b.upstream_rq_2xx: 173
cluster.server.zone.zone_a.zone_b.upstream_rq_completed: 173
$ docker-compose stop
```
