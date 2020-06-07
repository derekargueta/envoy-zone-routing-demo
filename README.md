Envoy Zone Routing Demo
=======================

This is some example code to accompany a blog post on the zone-aware routing abilities of Envoy.

The master branch contains a "direct-all-routing" scenario, where the percentage
of zone A clients 2/3 clients and 2/3 servers, so any traffic the zone A client
receives will stay in zone A.

## To run
```bash
$ git clone git@github.com:derekargueta/envoy-zone-routing-demo.git
$ cd envoy-zone-routing-demo
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
