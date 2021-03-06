![](images/prometheus_github.svg)

# Prometheus Training

    Marcus Teixeira
    https://github.com/marcusteixeira/prometheus-training

Source of my Prometheus Training

## About Course

- Prometheus 101 - Introduction to Prometheus and Cloud Native Applications

## Star, Create Issues, Fork, and Contribute

Feel free to star this repository or fork it.

If you found bug, create issue or pull request.

Also feel free to propose improvements by creating issues.

## Requirements

You need to have docker and docker-compose installed (e.g. by installing Docker Desktop <https://www.docker.com/products/docker-desktop)>

## Course

## Agenda

- Prometheus
  - Intro
  - Install Prometheus
  - Basic configuration
  - Scraping & Exporters
  - Push Gateway
  - PromQL
  - Alerting
- Alert Manager
  - Install Alert Manager
  - Routes
  - Receivers
- Grafana
  - Install Grafana
  - Working with dashboards
  - Prometheus integration

## What is Prometheus

Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company. To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes. -- [Prometheus website](https://prometheus.io/docs/introduction/overview/#what-is-prometheus)

### Prometheus Features

- time series DB
- PromQL - a flexible query language
- metrics scraping
- support for push metrics (Push Gateway)
- service discovery & static config
- many exporters
- alert manager

### Prometheus Architecture

![Prometheus Architecture](images/prometheus-architecture.png)


![Prometheus Architecture](images/prometheus_map.jpg)

### Metric Types

- Counter
- Gauge
- Histogram
- Summary

#### Counter

A counter is a cumulative metric that represents a single monotonically increasing counter whose value can only increase or be reset to zero on restart. For example, you can use a counter to represent the number of requests served, tasks completed, or errors.

#### Gauge

A gauge is a metric that represents a single numerical value that can arbitrarily go up and down.

Gauges are typically used for measured values like temperatures or current memory usage, but also "counts" that can go up and down, like the number of concurrent requests.

#### Histogram

A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values.

A histogram with a base metric name of `<basename>` exposes multiple time series during a scrape:

- cumulative counters for the observation buckets, exposed as `<basename>_bucket{le="<upper inclusive bound>"}`
- the total sum of all observed values, exposed as `<basename>_sum`
- the count of events that have been observed, exposed as `<basename>_count (identical to <basename>_bucket{le="+Inf"}` above)

## Run Prometheus

### Test Prometheus with Simple Config

Run

```
prometheus --config.file=01_prometheus.yml
```

See <http://127.0.0.1:9090>

![](images/prometheus-default-view.png)

### Run Random Metrics Generator

Run in Docker (see [source](docker/random-metrics))

```
docker run --name random8080 -d -p 8080:8080 ondrejsika/random-metrics
docker run --name random8081 -d -p 8081:8080 ondrejsika/random-metrics
docker run --name random8082 -d -p 8082:8080 ondrejsika/random-metrics
```

Run Prometheus with those sample targets

```
prometheus --config.file=02_prometheus.yml
```

See <http://127.0.0.1:9090>

## Prometeheus Exporters

### What are Prometeheus Exporters

There are a number of libraries and servers which help in exporting existing metrics from third-party systems as Prometheus metrics. This is useful for cases where it is not feasible to instrument a given system with Prometheus metrics directly (for example, HAProxy or Linux system stats).

#### Popular exporters

- Node Exporter (official) - <https://github.com/prometheus/node_exporter>
- Blackbox Exporter (official) - <https://github.com/prometheus/blackbox_exporter>
- cAdvisor (Docker) - <https://github.com/google/cadvisor>
- Kube State Metrics - <https://github.com/kubernetes/kube-state-metrics>
- MySQL Exporter (official) - <https://github.com/prometheus/mysqld_exporter>
- Postgres Exporter - <https://github.com/wrouesnel/postgres_exporter>
- WMI Exporter - <https://github.com/martinlindhe/wmi_exporter>

All exporters are on Prometheus website: <https://prometheus.io/docs/instrumenting/exporters/>
Defult ports of exporters: <https://github.com/prometheus/prometheus/wiki/Default-port-allocations>

### Node Exporter

[source](https://github.com/prometheus/node_exporter)

Install on host using Docker:

```
docker run --name node-exporter -d --net=host --pid=host -v /:/host:ro,rslave quay.io/prometheus/node-exporter --path.rootfs=/host
```

See: <https://node.demo.do.prometheus.io/metrics>

### Blackbox Exporter

[source](https://github.com/prometheus/blackbox_exporter)

Install on host using Docker:

```
docker run --rm -d -p 9115:9115 --name blackbox_exporter prom/blackbox-exporter:master
```

See: <http://example.sikademo.com:9115/metrics>

Check status code 200 on website:

- sika.io: <http://example.sikademo.com:9115/probe?module=http_2xx&target=https://sika.io>
- foo.int (not working): <http://example.sikademo.com:9115/probe?module=http_2xx&target=https://foo.int>


### cAdvisor

[source](https://github.com/google/cadvisor)

Install on host using Docker:

```
docker run --volume=/:/rootfs:ro --volume=/var/run:/var/run:ro --volume=/sys:/sys:ro --volume=/var/lib/docker/:/var/lib/docker:ro --volume=/dev/disk/:/dev/disk:ro --publish=9338:9338 --detach=true --name=cadvisor gcr.io/google-containers/cadvisor:latest --port=9338
```

See:

- Metrics: <http://example.sikademo.com:9338/metrics>
- Dashboard: <http://example.sikademo.com:9338/>

## PromQL

Select time series

### [Guide ] (<https://medium.com/@valyala/promql-tutorial-for-beginners-9ab455142085)>

```
node_network_receive_bytes_total
```

Select time series by label

```
node_network_receive_bytes_total{device="eth0"}
```

```
node_network_receive_bytes_total{device!="lo"}
```

Regular Expressions

```
node_network_receive_bytes_total{device=~"eth.+"}
```

```
node_network_receive_bytes_total{device!~"eth.+"}
```

```
node_network_receive_bytes_total{device=~"eth0|lo"}
```

Offset

```
node_network_receive_bytes_total offset 1h
```

Rates

```
rate(node_network_receive_bytes_total[5m])
```

### Examples

CPU usage in percent

```
100 * (1 - avg(rate(node_cpu_seconds_total{mode="idle"}[1m]))  by (instance))
```

Memory usage in percent

```
100 * (node_memory_Active_bytes / on (instance) node_memory_MemTotal_bytes)
```

Disk Usage in Percent

```
100 * (node_filesystem_avail_bytes{fstype!~"tmpfs|fuse.lxcfs|squashfs|vfat"} / node_filesystem_size_bytes{fstype!~"tmpfs|fuse.lxcfs|squashfs|vfat"})
```

Network transmit in kbps

```
sum(rate(node_network_transmit_bytes_total{device=~"eth.*|enp.*"}[10m])) by (instance)
```

### Saved Queries

```
prometheus --config.file=05_prometheus.yml
```


## Push Gateway

Install Push Gateway using Docker:

```
docker run --name push-gateway -d -p 9091:9091 prom/pushgateway
```

See:

- Web UI: <http://example.sikademo.com:9091/>
- Metrics: <http://example.sikademo.com:9091/metrics>

### Examples

Push with label `{job="some_job"}`

```
echo "demo 3.14" | curl --data-binary @- http://example.sikademo.com:9091/metrics/job/some_job
```

Push with label `{job="other_job",instance="some_instance"}`

```
cat <<EOF | curl --data-binary @- http://example.sikademo.com:9091/metrics/job/some_job/instance/some_instance
# TYPE some_metric counter
some_metric{label="val1"} 42
# TYPE another_metric gauge
# HELP another_metric Just an example.
another_metric 2398.283
EOF
```

Delete metrics from Push Gateway:

```
curl -X DELETE http://example.sikademo.com:9091/metrics/job/some_job
```

```
curl -X DELETE http://example.sikademo.com:9091/metrics/job/some_job/instance/some_instance
```

## Alertmanager

### Maildev

Run [maildev](https://github.com/maildev/maildev) on localhost:

```
docker run --name maildev -d -p 1080:80 -p 1025:25 djfarrelly/maildev
```

See: <http://localhost:1080>


### Run Alertmanager Example

Run Random Metrics:

```
docker run --name random8080 -d -p 8080:8080 ondrejsika/random-metrics
docker run --name random8081 -d -p 8081:8080 ondrejsika/random-metrics
docker run --name random8082 -d -p 8082:8080 ondrejsika/random-metrics
```

Run Prometheus with rules configuration

```
prometheus --config.file=03_prometheus.yml
```

See: <http://localhost:9090/alerts>

and in other tab run Alertmanager

```
alertmanager --config.file 03_alertmanager.yml
```

See: <http://localhost:9093>

Stop random metrics:

```
docker stop random8080 random8081 random8082
```

See Alerts, Alertmanager and Emaildev. Start them and check again:

```
docker start random8080 random8081 random8082
```

### Multiple Receivers Example

Use `./set-probe_success.sh` script to set everything up

```
./set-probe_success.sh frontend 1
./set-probe_success.sh backend 1
./set-probe_success.sh db 1
./set-probe_success.sh lb 1
```

Run Prometheus & Alertmanager:

```
prometheus --config.file=04_prometheus.yml
```

```
alertmanager --config.file 04_alertmanager.yml
```

You can debug routes here: <https://prometheus.io/webtools/alerting/routing-tree-editor/>

Check Alerts & Alert Manager.

Fire some errors:

```
./set-probe_success.sh db 0
./set-probe_success.sh lb 0
```

Check Alerts, Alert Manager & MailDev.

Fix DB & LB and see Alerts, Alert Manager & MailDev again.

```
./set-probe_success.sh db 1
./set-probe_success.sh lb 1
```

## Grafana

### Dashboards on Grafana.com

All dashboards are on: <https://grafana.com/grafana/dashboards>

My favourite dashboards:

- Node Exporter - `405` - <https://grafana.com/grafana/dashboards/405>
- Node Exporter - `11074` - <https://grafana.com/grafana/dashboards/11074>
- Postgres - `455` - <https://grafana.com/grafana/dashboards/455>
- Mysql - `6239` - <https://grafana.com/grafana/dashboards/6239>
- Traefik - `5851` - <https://grafana.com/grafana/dashboards/5851>
- Proxmox - `10347` - <https://grafana.com/grafana/dashboards/10347>

Kubernetes

- K8 Cluster Detail Dashboard - `10856` - <https://grafana.com/grafana/dashboards/10856>
- 10 Project/NameSpace Based on Memory - `10551` - <https://grafana.com/grafana/dashboards/10551>
- Cluster Monitoring for Kubernetes - `10000` - <https://grafana.com/grafana/dashboards/10000>

## Resources

- Prometheus vs Others - <https://prometheus.io/docs/introduction/comparison/>
- Prometheus Definitive Guide - <https://devconnected.com/the-definitive-guide-to-prometheus-in-2019/>
- Kubernertes Monitoring - <https://observability.thomasriley.co.uk/>
- Prometheus For Beginners - <https://itnext.io/prometheus-for-beginners-5f20c2e89b6c>

## Case Studies

- [Prometheus at Prezi: replacing 10 years of anti-patterns](https://link.medium.com/n7JMnn9dd3)
