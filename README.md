# 0G
The First Modular AI Chain



# Hardware Requirement [^1]
[^1]: [0G Docs Run Node Validator Node](https://docs.0g.ai/0g-doc/run-a-node/validator-node)

- CPU : 4 vCPU
- RAM : 8 GB
- Storage : 500 GB NVMe SSD
- Network : 100mbps Gbps for Download / Upload

> [!TIP]
> `CPU`, `Network` and `Disk I/O` become a potential bottleneck during High TPS.
> If could, it is better to choose higher spec server than requirement.


# Install 0G Validator node
Currently in production.  
Please use [Moderator Daniel Moon guide](https://github.com/trusted-point/0g-tools) as a reference.


# Set up monitoring system: Prometheus + Grafana
> [!TIP]
> **_1. Purpose_**  
> The purpose of system monitoring is to detect potential issues or anomalies in the system, so that they can be addressed proactively before they cause significant problems on network.
> 
> **_2. Prometheus_**  
> An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
>   
> **_3. Grafana_**  
> The open and composable observability and data visualization platform. Visualize metrics, logs, and traces from multiple sources like Prometheus and Loki.

> [!CAUTION]
> This guide is supported for the node by following the [Moderator Daniel Moon guide](https://github.com/trusted-point/0g-tools) guide.  
> Support OS is `Ubuntu22.04 LTS`.

## Stop 0G validator node
Run this command:
``` bash
sudo systemctl stop ogd
```


## Change gRPC port to 19090 from 9090
Run this command:
> [!TIP]
> 0G grpc and prometheus port number are same.
> To avoid port confliction, change 0G gRPC port to 19090.
``` bash
GRPC_PORT=19090

sudo sed -i \
    -e "/\[grpc\]/,/^\[/{s/\(address = \"\)\([^:]*\):\([0-9]*\)\(\".*\)/\1\2:$GRPC_PORT\4/}" $HOME/.evmosd/config/app.toml
```

## Enable Prometheus
Run this command:
``` bash
sudo sed -i "s/^prometheus *=.*/prometheus = true/" $HOME/.evmosd/config/config.toml
```

Check if `prometheus` is `true`:
``` bash
sudo grep prometheus $HOME/.evmosd/config/config.toml
```

**----- Result -----**
```
prometheus = true
prometheus_listen_addr = ":26660"
```


## Start 0G validator node
Run this command:
``` bash
sudo systemctl start ogd
```


## Install Prometheus

Run this command:
``` bash
sudo apt-get install -y prometheus prometheus-node-exporter
```

Check if `prometheus` and `prometheus-node-exporter` has been installed:
``` bash
sudo dpkg -l prometheus prometheus-node-exporter
```

**----- Result -----**
```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                     Version                     Architecture Description
+++-========================-===========================-============-==========================================
ii  prometheus               2.31.2+ds1-1ubuntu1.22.04.2 amd64        monitoring system and time series database
ii  prometheus-node-exporter 1.3.1-1ubuntu0.22.04.2      amd64        Prometheus exporter for machine metrics
```

> [!TIP]
> The command install both `Prometheus` and `Prometheus Node Exporter`.  
> The `Prometheus Node Exporter` exposes a wide variety of `hardware-` and `kernel-related` metrics. [^2]
[^2]: [Prometheus: MONITORING LINUX HOST METRICS WITH THE NODE EXPORTER](https://prometheus.io/docs/guides/node-exporter/)

## Install Grafana
Run this command:
``` bash
sudo wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo echo "deb https://packages.grafana.com/oss/deb stable main" > grafana.list
sudo mv grafana.list /etc/apt/sources.list.d/grafana.list

sudo apt-get update && sudo apt-get install -y grafana
```

Check if `grafana` has been installed:
``` bash
sudo dpkg -l grafana
```

**----- Result -----**
```
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version      Architecture Description
+++-==============-============-============-=================================
ii  grafana        10.2.3       amd64        Grafana
```

## Start Prometheus and Grafana
Run this command:
``` bash
sudo systemctl enable grafana-server.service prometheus.service prometheus-node-exporter.service
sudo systemctl start grafana-server.service prometheus.service prometheus-node-exporter.service
```

1. Check if `Grafana` status is `active (running)`:
``` bash
sudo systemctl status grafana-server.service --no-pager -l
```

**----- Result -----**
```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: http://docs.grafana.org
   Main PID: 69210 (grafana)
      Tasks: 21 (limit: 76910)
     Memory: 50.5M
        CPU: 2min 1.834s
     CGroup: /system.slice/grafana-server.service
             └─69210 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/l…

Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:05:26.769563461+01:00 level=info msg="Update check succeeded" duration=7.663637ms
Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:05:26.808206779+01:00 level=info msg="Completed cleanup jobs" duration=64.861523ms
Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:05:26.874721247+01:00 level=info msg="Update check succeeded" duration=70.901991ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:15:26.769039867+01:00 level=info msg="Update check succeeded" duration=6.935579ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:15:26.810085388+01:00 level=info msg="Completed cleanup jobs" duration=66.294104ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:15:26.875407205+01:00 level=info msg="Update check succeeded" duration=72.228362ms
Dec 27 11:16:42 0g-node-testnet grafana[69210]: logger=infra.usagestats t=2023-12-27T11:16:42.755259475+01:00 level=info msg="Usage stats are ready to report"
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:25:26.768865417+01:00 level=info msg="Update check succeeded" duration=6.73339ms
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:25:26.809052346+01:00 level=info msg="Completed cleanup jobs" duration=65.2849ms
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:25:26.860347684+01:00 level=info msg="Update check succeeded" duration=56.999845ms
```

2. Check if `Prometheus` status is `active (running)`:
``` bash
sudo systemctl status prometheus.service --no-pager -l
```

**----- Result -----**
```
● prometheus.service - Monitoring system and time series database
     Loaded: loaded (/lib/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:19 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://prometheus.io/docs/introduction/overview/
             man:prometheus(1)
   Main PID: 69220 (prometheus)
      Tasks: 18 (limit: 76910)
     Memory: 130.9M
        CPU: 4min 19.124s
     CGroup: /system.slice/prometheus.service
             └─69220 /usr/bin/prometheus

Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.716Z caller=compact.go:459 level=info component=tsdb msg="compact blocks" count=3 mint=1703613603258 maxt=1703635200000 ulid=01HJMT9WENTPNRFKS8V7AV04KN sources="[01HJKYTZ67785VT052NA9T7P1T 01HJM5PHHZPQ3NF1QEQFQ5PGAF 01HJMCJ8T01V9EDKPT5QVACB1S]" duration=183.420025ms
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.725Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJKYTZ67785VT052NA9T7P1T
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.731Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJM5PHHZPQ3NF1QEQFQ5PGAF
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.740Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJMCJ8T01V9EDKPT5QVACB1S
Dec 27 08:00:03 0g-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.400Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703649603257 maxt=1703656800000 ulid=01HJN15EHZ5AMEWV02TT0K5SM0 duration=136.587432ms
Dec 27 08:00:03 0g-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.402Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.514546ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.421Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703656803257 maxt=1703664000000 ulid=01HJN81AP7G92NC8F4CHQP4X60 duration=157.185812ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.423Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.538992ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.452Z caller=checkpoint.go:97 level=info component=tsdb msg="Creating checkpoint" from_segment=12 to_segment=13 mint=1703664000000
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.541Z caller=head.go:974 level=info component=tsdb msg="WAL checkpoint complete" first=12 last=13 duration=88.867604ms
```


3. Check if `Prometheus Node Exporter` status is `active (running)`:
``` bash
sudo systemctl status prometheus-node-exporter.service --no-pager -l
```

**----- Result -----**
```
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
     Loaded: loaded (/lib/systemd/system/prometheus-node-exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://github.com/prometheus/node_exporter
   Main PID: 69211 (prometheus-node)
      Tasks: 29 (limit: 76910)
     Memory: 13.7M
        CPU: 11min 42.505s
     CGroup: /system.slice/prometheus-node-exporter.service
             └─69211 /usr/bin/prometheus-node-exporter

Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=thermal_zone
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=time
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=timex
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=udp_queues
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=uname
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=vmstat
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=xfs
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=zfs
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```

## Add 0G metrics target to Prometheus config file
Run this command:
```
sudo sh -c "cat << EOT >> /etc/prometheus/prometheus.yml

  - job_name: 0g
    static_configs:
      - targets: ['localhost:26660']
EOT"
```


Check `Prometheus config file`:
``` bash
sudo cat /etc/prometheus/prometheus.yml
```
**----- Result -----**
``` yaml
# Sample config for Prometheus.

global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 0g        # <======================== CHECK HERE
    static_configs:     # <======================== CHECK HERE
      - targets: ['localhost:26660']     # <======= CHECK HERE
```


## Reload Prometheus
Run this command:
``` bash
sudo systemctl reload prometheus.service
```

## Restart Prometheus and Grafana
Run this command:
``` bash
sudo systemctl restart grafana-server.service prometheus.service prometheus-node-exporter.service
```

1. Check if `Grafana` status is `active (running)`:
``` bash
sudo systemctl status grafana-server.service --no-pager -l
```

**----- Result -----**
```
● grafana-server.service - Grafana instance
     Loaded: loaded (/lib/systemd/system/grafana-server.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: http://docs.grafana.org
   Main PID: 69210 (grafana)
      Tasks: 21 (limit: 76910)
     Memory: 50.5M
        CPU: 2min 1.834s
     CGroup: /system.slice/grafana-server.service
             └─69210 /usr/share/grafana/bin/grafana server --config=/etc/grafana/grafana.ini --pidfile=/run/grafana/grafana-server.pid --packaging=deb cfg:default.paths.logs=/var/l…

Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:05:26.769563461+01:00 level=info msg="Update check succeeded" duration=7.663637ms
Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:05:26.808206779+01:00 level=info msg="Completed cleanup jobs" duration=64.861523ms
Dec 27 11:05:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:05:26.874721247+01:00 level=info msg="Update check succeeded" duration=70.901991ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:15:26.769039867+01:00 level=info msg="Update check succeeded" duration=6.935579ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:15:26.810085388+01:00 level=info msg="Completed cleanup jobs" duration=66.294104ms
Dec 27 11:15:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:15:26.875407205+01:00 level=info msg="Update check succeeded" duration=72.228362ms
Dec 27 11:16:42 0g-node-testnet grafana[69210]: logger=infra.usagestats t=2023-12-27T11:16:42.755259475+01:00 level=info msg="Usage stats are ready to report"
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=grafana.update.checker t=2023-12-27T11:25:26.768865417+01:00 level=info msg="Update check succeeded" duration=6.73339ms
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=cleanup t=2023-12-27T11:25:26.809052346+01:00 level=info msg="Completed cleanup jobs" duration=65.2849ms
Dec 27 11:25:26 0g-node-testnet grafana[69210]: logger=plugins.update.checker t=2023-12-27T11:25:26.860347684+01:00 level=info msg="Update check succeeded" duration=56.999845ms
```

2. Check if `Prometheus` status is `active (running)`:
``` bash
sudo systemctl status prometheus.service --no-pager -l
```

**----- Result -----**
```
● prometheus.service - Monitoring system and time series database
     Loaded: loaded (/lib/systemd/system/prometheus.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:19 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://prometheus.io/docs/introduction/overview/
             man:prometheus(1)
   Main PID: 69220 (prometheus)
      Tasks: 18 (limit: 76910)
     Memory: 130.9M
        CPU: 4min 19.124s
     CGroup: /system.slice/prometheus.service
             └─69220 /usr/bin/prometheus

Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.716Z caller=compact.go:459 level=info component=tsdb msg="compact blocks" count=3 mint=1703613603258 maxt=1703635200000 ulid=01HJMT9WENTPNRFKS8V7AV04KN sources="[01HJKYTZ67785VT052NA9T7P1T 01HJM5PHHZPQ3NF1QEQFQ5PGAF 01HJMCJ8T01V9EDKPT5QVACB1S]" duration=183.420025ms
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.725Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJKYTZ67785VT052NA9T7P1T
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.731Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJM5PHHZPQ3NF1QEQFQ5PGAF
Dec 27 06:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T05:00:08.740Z caller=db.go:1293 level=info component=tsdb msg="Deleting obsolete block" block=01HJMCJ8T01V9EDKPT5QVACB1S
Dec 27 08:00:03 0g-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.400Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703649603257 maxt=1703656800000 ulid=01HJN15EHZ5AMEWV02TT0K5SM0 duration=136.587432ms
Dec 27 08:00:03 0g-node-testnet prometheus[69220]: ts=2023-12-27T07:00:03.402Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.514546ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.421Z caller=compact.go:518 level=info component=tsdb msg="write block" mint=1703656803257 maxt=1703664000000 ulid=01HJN81AP7G92NC8F4CHQP4X60 duration=157.185812ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.423Z caller=head.go:805 level=info component=tsdb msg="Head GC completed" duration=1.538992ms
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.452Z caller=checkpoint.go:97 level=info component=tsdb msg="Creating checkpoint" from_segment=12 to_segment=13 mint=1703664000000
Dec 27 10:00:08 0g-node-testnet prometheus[69220]: ts=2023-12-27T09:00:08.541Z caller=head.go:974 level=info component=tsdb msg="WAL checkpoint complete" first=12 last=13 duration=88.867604ms
```


3. Check if `Prometheus Node Exporter` status is `active (running)`:
``` bash
sudo systemctl status prometheus-node-exporter.service --no-pager -l
```

**----- Result -----**
```
● prometheus-node-exporter.service - Prometheus exporter for machine metrics
     Loaded: loaded (/lib/systemd/system/prometheus-node-exporter.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-12-26 04:15:18 CET; 1 day 7h ago       <===== CHECK HERE
       Docs: https://github.com/prometheus/node_exporter
   Main PID: 69211 (prometheus-node)
      Tasks: 29 (limit: 76910)
     Memory: 13.7M
        CPU: 11min 42.505s
     CGroup: /system.slice/prometheus-node-exporter.service
             └─69211 /usr/bin/prometheus-node-exporter

Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=thermal_zone
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=time
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=timex
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=udp_queues
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=uname
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=vmstat
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=xfs
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:115 level=info collector=zfs
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=node_exporter.go:199 level=info msg="Listening on" address=:9100
Dec 26 04:15:18 0g-node-testnet prometheus-node-exporter[69211]: ts=2023-12-26T03:15:18.990Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false
```



# Set up Grafana Dashboard
> [!WARNING]
> If you set `ufw` at your server, You should open Grafana port (default: `3000`).  
> `sudo ufw allow 3000/tcp`

> [!IMPORTANT]
> The following steps should be executed from your **Client, NOT server**.  


## Access your Grafana
Launch browser and go to `http://<YOUR-SERVER-GLOBAL-IP>:3000/`

> [!TIP]
> Run this command to check your server global ip.  
> `curl inet-ip.info`
<img width="1680" alt="grafana-01" src="https://github.com/qyeah98/voi/assets/99518712/123d7194-bd1c-45a6-9870-bbd331c91683">


## Login to Grafana
> [!TIP]
> Default username / password is `admin` / `admin`
<img width="1680" alt="grafana-02" src="https://github.com/qyeah98/voi/assets/99518712/c8d5a30e-cb02-41f9-8fd7-964e8367be91">

## Go to Connections -> Data Sources
<img width="1680" alt="grafana-04" src="https://github.com/qyeah98/voi/assets/99518712/4b8d5f57-9d46-4385-8596-9a84dacf7d65">

## Click Add new data source and select Prometheus
<img width="1680" alt="grafana-05" src="https://github.com/qyeah98/voi/assets/99518712/6f0a32c7-e952-4090-bf0e-90e9988165ef">
<img width="1680" alt="grafana-06" src="https://github.com/qyeah98/voi/assets/99518712/a0bb03fe-cdc4-4a19-9cf5-ebe0a2989960">

## Enter http://localhost:9090 into Prometheus server URL
<img width="1680" alt="grafana-07" src="https://github.com/qyeah98/voi/assets/99518712/b98ff99e-08cd-4262-b49f-206f8cc0f135">

## Scroll to the bottom and click Save & test
<img width="1680" alt="grafana-08" src="https://github.com/qyeah98/voi/assets/99518712/f473c4ad-7eae-48c1-b8d1-e1c682d90a5a">

## Go to Dashboards to create dashboard
<img width="1680" alt="grafana-10" src="https://github.com/qyeah98/voi/assets/99518712/a167cf36-7df2-41d8-987f-65b9020fdc0f">
<img width="1680" alt="grafana-11" src="https://github.com/qyeah98/voi/assets/99518712/3276da80-3d04-4ee8-8d89-eddaa02711f9">

## Click New > Import
<img width="1680" alt="grafana-12" src="https://github.com/qyeah98/voi/assets/99518712/5be3761c-b4fa-4b7a-aa8a-9ccd048dbe48">

## Click Upload JSON file
Please download the [0G grafana json file](https://github.com/qyeah98/0g/blob/main/grafana/0g-node.json) and upload it.
<img width="1680" alt="grafana-13" src="https://github.com/qyeah98/voi/assets/99518712/45d6e169-4fba-4468-b2de-b6b523312fbb">

## Select prometheus datasource
<img width="1680" alt="grafana-15" src="https://github.com/qyeah98/voi/assets/99518712/877795fb-ef44-4c65-a510-379db89c5734">
<img width="1680" alt="grafana-16" src="https://github.com/qyeah98/voi/assets/99518712/be5cfd78-46f2-4935-bbe8-cd5b37e1a06e">

## Complete
<img width="1680" alt="grafana-17" src="https://github.com/qyeah98/voi/assets/99518712/2714276f-54e9-43cf-bdcd-b4a79dda0c5a">

# (Optional) Set up notification system: Grafana + PagerDuty
> [!TIP]
> **_PagerDuty_**  
> PagerDuty can automatically group related alerts into a single incident to minimize noise while centralizing relevant context.
> Incident notifications are automatically sent using any preferred combination of phone calls, SMS, push notifications and emails.

> [!CAUTION]
> `Free plan` does not support international `phone` / `SMS` notifications.  
> If you want to receive notification over call or SMS, you have to subscribe `Professional plan` ($ 21 per user/month) [^8]  
> This guide does not support how to sign up PagerDuty.  
> If you does not have the account, please sign up in advance.  
>   
> You do not need to sign up PagerDuty to receive alert at `Slack` / `Discord` / `Telegram`.  
> Please skip "Generate PagerDuty API Key" selction.
[^8]: [PagerDuty Plan and pricing](https://www.pagerduty.com/pricing/incident-response/)

## Generate PagerDuty API Key
### Go to Service Directory
<img width="1680" alt="pagerduty-01" src="https://github.com/qyeah98/voi/assets/99518712/46eef1be-a23f-4173-97b8-6139d8160df7">

### Click +New Service
<img width="1680" alt="pagerduty-02" src="https://github.com/qyeah98/voi/assets/99518712/61de7bd2-3818-4efd-a1a4-b059b8486ac9">

### Input Service Name
<img width="1680" alt="pagerduty-03" src="https://github.com/qyeah98/voi/assets/99518712/614bef36-dc58-4ddb-b22c-3cf8cf376bac">

### Select Escalation Policy
> [!TIP]
> If you want to assign existing escalation rule, please click "Select an existing Escalation Policy".
<img width="1680" alt="pagerduty-04" src="https://github.com/qyeah98/voi/assets/99518712/7cc19224-5dc4-4389-b8b7-ced62aedcc73">

### Reduce Noise
> [!TIP]
> Not every alert should be an incident.  
> PagerDuty’s noise reduction capabilities leverage data science and machine learning to cut down on system noise and reduce alert fatigue.  
> This means fewer incidents and interruptions for on-call responders, richer context around the incidents that do trigger, and lower resolution times.
<img width="1680" alt="pagerduty-05" src="https://github.com/qyeah98/voi/assets/99518712/88f08a20-f8d8-4c13-a98b-97daf2ea69e6">
<img width="1680" alt="pagerduty-06" src="https://github.com/qyeah98/voi/assets/99518712/54ce32fe-c90e-4e3f-9eac-be4c6d1ad710">

### Select integration service
<img width="1680" alt="pagerduty-07" src="https://github.com/qyeah98/voi/assets/99518712/1e77f89c-621e-4752-89da-f0001c775883">
<img width="1680" alt="pagerduty-08" src="https://github.com/qyeah98/voi/assets/99518712/377d20d4-4735-4cc6-9ce4-b46323902062">

### Copy Integration Key
<img width="1680" alt="pagerduty-09" src="https://github.com/qyeah98/voi/assets/99518712/03ef61a0-73c6-40d9-a7c7-c60b1848c2bf">

## Integration Grafana with PagerDuty
### Access your Grafana
Launch browser and go to `http://<YOUR-SERVER-GLOBAL-IP>:3000/`

### Go to Alerting > Contact point
<img width="1680" alt="pagerduty-10" src="https://github.com/qyeah98/voi/assets/99518712/28e245e2-48b3-499f-802f-2fb0da8481f6">
<img width="1680" alt="pagerduty-11" src="https://github.com/qyeah98/voi/assets/99518712/03dff62c-eefa-4250-aa97-13f4fc94bcd2">


### Add Contact Point
<img width="1680" alt="pagerduty-12" src="https://github.com/qyeah98/voi/assets/99518712/9cf9ef03-c65e-4ddf-b382-4c25619bba2d">

> [!TIP]
> When click "Test", you'll receive notification from PagerDuty.  
> If you'd like to change on-call way, please change notification rule at PagerDuty. [^9]  
>  
> If you want to receive alert at `Slack` / `Discord` / `Telegram`, please select `Slack` / `Discord` / `Telegram`
[^9]: [PagerDuty User Profile](https://support.pagerduty.com/docs/user-profile#:~:text=Edit%20a%20Notification%20Rule,-Edit%20any%20notification&text=In%20the%20web%20app%2C%20navigate,you'd%20like%20to%20change.)

<img width="1680" alt="pagerduty-13" src="https://github.com/qyeah98/voi/assets/99518712/6313344c-2b07-4ba2-9c10-7df1c447e6dc">


### Change Grafana Notification policies
<img width="1680" alt="pagerduty-14" src="https://github.com/qyeah98/voi/assets/99518712/2ddb6765-f943-4b52-94d1-d1965a4d7218">
<img width="1680" alt="pagerduty-15" src="https://github.com/qyeah98/voi/assets/99518712/9ba7c55b-002e-43a2-aff9-faa8e663407b">
<img width="1680" alt="pagerduty-16" src="https://github.com/qyeah98/voi/assets/99518712/821cf4db-85f1-4b20-8aa8-6caa51927cf3">

### Go to 0G Node Dashboard
<img width="1680" alt="pagerduty-17" src="https://github.com/qyeah98/voi/assets/99518712/f06919c3-8979-4616-b75b-fd66c8de3e0e">
<img width="1680" alt="pagerduty-18" src="https://github.com/qyeah98/voi/assets/99518712/55fad085-ad66-47a1-8242-449fc637934e">

### Creating Alert Rule
> [!TIP]
> _Q. What is Grafana Alerting Rule?_  
> A. An alert rule consists of one or more queries and expressions, a condition, and the duration over which the condition needs to be met to start firing.  
> While queries and expressions select the data set to evaluate, a condition sets the threshold that an alert must meet or exceed to create an alert.

> [!IMPORTANT]
> What metrics you choose is the most important thing to monitoring your node.  
> This guide is monitoring `Total Peers` metrics to notify when `peer is 0`.  
> I don't have best procatice, so let's discuss.  
<img width="1680" alt="pagerduty-19" src="https://github.com/qyeah98/voi/assets/99518712/55258da1-c7e1-4c1b-9311-0da841ecfcec">
<img width="1680" alt="pagerduty-20" src="https://github.com/qyeah98/voi/assets/99518712/92d96bc6-a1bf-421a-9fe4-5fc34f4d6e53">
<img width="1680" alt="pagerduty-21" src="https://github.com/qyeah98/voi/assets/99518712/34712553-bf14-40cd-a691-b982106d05e0">
<img width="1680" alt="pagerduty-22" src="https://github.com/qyeah98/voi/assets/99518712/b40544c0-db19-4681-9f63-2b0201e4e7e4">
<img width="1680" alt="pagerduty-23" src="https://github.com/qyeah98/voi/assets/99518712/416868fb-365e-4980-814c-ba1fbebd9bf3">
<img width="1680" alt="pagerduty-24" src="https://github.com/qyeah98/voi/assets/99518712/c1e144d6-4756-4e82-9885-72ce5ff85308">
<img width="1680" alt="pagerduty-25" src="https://github.com/qyeah98/voi/assets/99518712/eef8e7c8-1c52-4563-bdff-135cc484d0a2">
<img width="1680" alt="pagerduty-26" src="https://github.com/qyeah98/voi/assets/99518712/d27af341-de09-4787-82fd-9e8798bc8e20">
<img width="1680" alt="pagerduty-27" src="https://github.com/qyeah98/voi/assets/99518712/34fd8296-0af5-4ffc-8d8a-5e2f1b21675b">
<img width="1680" alt="pagerduty-28" src="https://github.com/qyeah98/voi/assets/99518712/1d225b95-436a-4004-a24e-71e9ac87e348">

### Complete
> [!TIP]
> You can find alerting status at Dashboard.
<img width="1680" alt="pagerduty-29" src="https://github.com/qyeah98/voi/assets/99518712/7602b9b4-67c2-4954-bc9c-48f0b3861d64">

> [!IMPORTANT]
> To Keep network healhy, it is the important to detect aberrations in node metrics as soon as possible.  
> If you set monitor and notification, it lead to make network robustness.  
> I believe this is the responsibility for node operators.
