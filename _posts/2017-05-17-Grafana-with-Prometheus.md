---
layout: post
title: Grafana with Prometheus
feature-img: "img/sample_feature_img.png"
<<<<<<< HEAD
tag: [Grafana, Dock]
=======
tag: [Grafana, Prometheus, Docker]
>>>>>>> 8e17e637b8045e29623e73e185e6755d246e7ec2
---


1. Install Grafana, using docker

2. Install Prometheus, using docker 

   ```shell
   docker run -p 9090:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml /opt/prometheus/prometheus:/etc/prometheus/prometheus --name prometheus prom/prometheus -alertmanager.url=http://172.22.249.18:9093 -config.file=/etc/prometheus/prometheus.yml
   ```

3. install scollect and prometheus_scollector to node (windows OS)

   require: Golang

   ```shell
   go get bosun.org/scollector
   go get github.com/tgulacsi/prometheus_scollector
   prometheus_scollector -http=0.0.0.0:9107
   scollector -h=<the-collector-machine>:9107
   ```

1. add config to promethueus config



ps. disable w32time service (關閉時間同步)

## Blackbox-exporter

HTTP or TCP port 監控

```shell
docker build -t blackbox_exporter .
docker run -d -p 9115:9115 --name blackbox_exporter -v `pwd`:/config blackbox_exporter -config.file=/config/blackbox.yml
```

blackbox.yml example

```yaml
  http_2xx:
    prober: http
    timeout: 5s
    http:
      valid_status_codes: []  # Defaults to 2xx
      method: GET
      headers:
        Host: vhost.example.com
        Accept-Language: en-US
      no_follow_redirects: false
      fail_if_ssl: false
      fail_if_not_ssl: false
      fail_if_matches_regexp:
      - "Could not connect to database"
      fail_if_not_matches_regexp:
      - "Download the latest version here"
  tcp_connect:
    prober: tcp
    timeout: 5s
  pop3s_banner:
    prober: tcp
    tcp:
      query_response:
      - expect: "^+OK"
      tls: true
      tls_config:
        insecure_skip_verify: false
  ssh_banner:
    prober: tcp
    timeout: 5s
    tcp:
      query_response:
      - expect: "^SSH-2.0-"
  irc_banner:
    prober: tcp
    timeout: 5s
    tcp:
      query_response:
      - send: "NICK prober"
      - send: "USER prober prober prober :prober"
      - expect: "PING :([^ ]+)"
        send: "PONG ${1}"
      - expect: "^:[^ ]+ 001"
  icmp:
    prober: icmp
    timeout: 5s
```

Prometheus config

```yaml
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    target_groups:
      - targets:
        - 172.22.253.81:9080/
        - 172.22.253.85:9080/
        - 172.22.253.26:8080/
        - 172.22.253.26:8080/
        - 172.22.250.79:8080/
        - 172.22.250.177/
        
    relabel_configs:
      - source_labels: [__address__]
        regex: (.*)(:80)?
        target_label: __param_target
        replacement: ${1}

      - source_labels: [__param_target]
        regex: (.*)
        target_label: instance
        replacement: ${1}

      - source_labels: [] 
        regex: .*
        target_label: __address__
        replacement: 172.22.249.18:9115  # Blackbox exporter.

  - job_name: 'blackbox-tcp'
    scrape_interval: 10s
    metrics_path: /probe
    params:
      module: [tcp_connect]
    target_groups:
      - targets:
        - 172.22.253.81:32782
        - 172.22.253.81:32784
        - 172.22.253.81:32785 
        - 172.22.253.81:50000 
        - 172.22.253.81:1414
        - 172.22.253.85:9080
        - 172.22.253.85:50000 
        - 172.22.253.18:1414 

    relabel_configs:
      - source_labels: [__address__]
        regex: (.*)
        target_label: __param_target
        replacement: ${1}

      - source_labels: [__param_target]
        regex: (.*)
        target_label: instance
        replacement: ${1}

      - source_labels: []
        regex: .*
        target_label: __address__
        replacement: 172.22.249.18:9115  # Blackbox exporter.
```

## node_exporter

Linux  性能收集器 
預設 port: 9100

```shell
$ go get github.com/prometheus/node_exporter
$ sudo ln -s bin/node_exporter /usr/bin
```

設定為 service 開機時自動啟動 (Ubuntu)
```shell
$ sudo vi /etc/init/node_exporter.conf
```
node_export.conf 內容
```shell
start on startup
script
/usr/bin/node_exporter
end script
```
建立symbol link to init.d
```shell
$ sudo ln -s /etc/init/node_exporter.conf /etc/init.d/node_exporter
```
啟動service
```shell
$ service node_exporter start
```


