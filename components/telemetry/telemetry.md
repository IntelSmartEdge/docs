```text
SPDX-License-Identifier: Apache-2.0
Copyright (c) 2021 Intel Corporation
```

# Intel® Smart Edge Open Telemetry Documentation

## Overview
Intel® Smart Edge Open comes with a set of telemetry components, providing user with the ability to monitor the performance and health of the cluster nodes. This support allows users to retrieve information about the platform, the underlying hardware, cluster, and applications deployed. The data gathered by telemetry can be used to visualize metrics and resource consumption, set up alerts for certain events, and aid in making scheduling decisions based on the received telemetry.

Currently, the support for telemetry is focused on metrics; support for application tracing telemetry is planned in the future.


Telemetry bundle is installed by the following experience kits:
- [Developer Experience Kit]()
- [Private Wireless Experience Kit]()

## How It Works

The telemetry components used in Intel® Smart Edge Open are deployed from the Edge Controller as Kubernetes* (K8s) pods. Components for telemetry support include:

- collectors
- metric aggregators
- monitoring and visualization tools

### How it's deployed

Depending on the role of the component, it is deployed as either a Deployment or Deamonset. Generally, global components receiving inputs from local collectors are deployed as a Deployment type with a single replica set, whereas local collectors running on each host are deployed as Daemonsets. Local collectors running on Edge Nodes that collect platform metrics are deployed as privileged containers, using host networking. Communication between telemetry components is secured with TLS either using native TLS support for a given feature or using a reverse proxy running in a pod as a container. All the components are deployed as Helm charts.
<br/>
<br/>
<img style="display: block; margin: auto;" alt="21.09 Telemetry overview" src="images/telemetry_2109.svg">


The deployment of telemetry components in Intel® Smart Edge Open is easily configurable from the open-developer-experience-kits (DEK). The deployment of the Grafana dashboard is optional (telemetry_grafana_enable enabled by default). There are four distinctive flavors for the deployment of the CollectD collector, enabling the respective set of plugins (telemetry_flavor):

- common (default)
- flexran
- smartcity
- corenetwork

Further information on what plugins each flavor enables can be found in the Collectd section. All flags can be changed in ESP provisioning configuration file (before Smart Edge Open deployment):

- generate a custom configuration file with `./dek_provision.py --init-config > custom.yml`
- edit generated file and set telemetry flavor under `group vars: all:`, e.g.

```yaml
profiles:
  - name: SEO_DEK
    [...]
    group_vars:
      groups:
        all:
          telemetry_flavor: common
          telemetry_grafana_enable: true
          telemetry_prometheus_enable: true
          telemetry_statsd_exporter_enable: true
          telemetry_statsd_exporter_udp_port: 8125
          telemetry_statsd_exporter_tcp_port: 8125
          telemetry_collectd_enable: true
          telemetry_cadvisor_enable: true
```
- use the custom configuration for all the following `dek_provision.py` command invocations, i.e. `./dek_provision.py --config=custom.yml [...]`

### Collectd

Collectd is a daemon/collector enabling the collection of hardware metrics from computers and network equipment. It provides support for Collectd plugins, which extends its functionality for specific metrics collection such as Intel® PMU, and ovs-dpdk. The metrics collected are exposed to the Prometheus monitoring tool via the [CollectdExporter](https://github.com/prometheus/collectd_exporter) sidecar. In Intel® Smart Edge Open, Collectd is supported with the help of the OPNFV Barometer project - using its Docker image and available plugins. In Intel® Smart Edge Open, the Collectd pod is deployed as a K8s Daemonset on every available Edge Node, and it is deployed as a privileged container running in host network.

In DEK metrics obtained by NodeExporter are subject to `collectd_<metric_name>` pattern.

#### Plugins

There are four distinct sets of plugins (flavors) enabled for CollectD deployment that can be used depending on the use-case/workload being deployed on Intel® Smart Edge Open. `Common` is the default flavor in Intel® Smart Edge Open. The flavors available are: `common`, `corenetwork`, `flexran`, and `smartcity`. Below is a table specifying which CollectD plugins are enabled for each flavor.
The various CEEK flavors are enabled for CollectD deployment as follows:


| Common           | Core Network      | FlexRAN           | SmartCity          |
|------------------|:-----------------:|------------------:|-------------------:|
| cpu              | cpu               | cpu               | cpu                |
| cpufreq          | cpufreq           | cpufreq           | cpufreq            |
| load             | load              | load              | load               |
| hugepages        | hugepages         | hugepages         | hugepages          |
| intel_pmu        | intel_pmu         | intel_pmu         | intel_pmu          |
| ipmi             | ipmi              | ipmi              | ipmi               |
| network          | network           | network           | network            |
|                  | ovs_pmd_stats     | ovs_stats         | ovs_pmd_stat       |
|                  | ovs_stats         |                   |                    |

#### Specifying logging threshold

By default only logs of "WARNING" and above thresholds will be displayed. User can change this changing the variable `telemetry_collectd_log_level` to one of "INFO","WARNING","ERROR". Collectd is not built to support "DEBUG" level logs. Please be wary that setting this variable to lower threshold (like INFO) will result in increased disk usage.

#### Collectd collection interval

Due to Collectd architecture relying on Prometheus exporter, Collectd collection interval is half of polling interval of Prometheus set via `telemetry_prometheus_scrape_interval_seconds` variable. For more information please refer to Prometheus' documentation.
### NodeExporter

Node Exporter is a Prometheus exporter that exposes hardware and OS metrics of \*NIX kernels. The metrics are gathered within the kernel and exposed on a web server so they can be scraped by Prometheus. In Intel® Smart Edge Open, the Node Exporter pod is deployed as a K8s Daemonset; it is a privileged pod, using host networking that runs on every Edge Node in the cluster. It is enabled by default by DEK.

Node exporter is configured to expose the [default set of collectors and metrics](https://github.com/prometheus/node_exporter/tree/v1.0.0-rc.0#enabled-by-default). Support for distinct collector and metrics vary depending on Operating System and hardware configuration - for details refer to Node Exporter specification.

In Intel® Smart Edge Open metrics obtained by NodeExporter are subject to `node_<metric_name>` pattern.

### Cadvisor

The cAdvisor is a running daemon that provides information about containers running in the cluster. It collects and aggregates data about running containers such as resource usage, isolation parameters, and network statistics. Once the data is processed, it is exported. The data can be easily obtained by metrics monitoring tools such as Prometheus. In Intel® Smart Edge Open, cAdvisor is deployed as a K8s daemonset on every Edge Node and scraped by Prometheus. Cadvisor deployment can be disabled by setting the `telemetry_cadvisor_enable` flag to `false` in provisioning configuration file.

#### Cadvisor and performance

CAdvisor is often reported to consume large portions of resources. For deployments with heavy resource constraints, where the availability of container related metrics is not paramount, it is advised to disable it.

### StatsD

[StatsD](https://github.com/statsd/statsd#statsd---) is a network daemon which listens for statistics like counters and timers sent over UDP or TCP. Usage of StatsD exporter allows to convert received StatsD-style metrics to Prometheus metrics and export them. By default, any non-alphanumeric characters, including periods, is translated into underscores. There is also a possibility to define customized mapping for selected metrics and personalized labels. Mapping can be configured in `statsd:mappingConfig` section in `roles/telemetry/statsd-exporter/templates/values.yml.j2` helm chart configuration file. Example mapping configuration can be found in [StatsD exporter repository](https://github.com/prometheus/statsd_exporter#metric-mapping-and-configuration). StatsD exporter pod is deployed on a control plane as a K8s `Deployment` which receives StatsD metrics from pods and is scraped by Prometheus. It is enabled by default in DEK and can be enabled/disabled by changing the `telemetry_statsd_exporter_enable` flag (in provisioning configuration file).

### Prometheus

Prometheus is an open-source, community-driven toolkit for systems monitoring and alerting. The main features include:

- PromQL query language
- multi-dimensional, time-series data model
- support for dashboards and graphs

The main idea behind Prometheus is that it defines a unified metrics data format that can be hosted as part of any application that incorporates a simple web server. The data can be then scraped (downloaded) and processed by Prometheus using a simple HTTP/HTTPS connection.

In Intel® Smart Edge Open, Prometheus is deployed as a K8s Deployment with a single pod/replica on the EdgeNode. It is configured out of the box to scrape all other telemetry endpoints/collectors enabled in Intel® Smart Edge Open and gather data from them. Prometheus is enabled in the DEK by default with the telemetry/prometheus role.

#### Prometheus' collection interval

By default Prometheus' collection interval is set to 60 second. This value can be changed via `telemetry_prometheus_scrape_interval_seconds` variable. Prometheus exporters query metrics on demand, with notable exception of Collectd, where the exporter works as a cache for metrics provided by collectd. In order to make sure that Prometheus query always yields updated metrics the interval of Collectd was decreased to match half of Prometheus' polling interval.

#### Prometheus' retention setting

Prometheus is not long-term storage for metrics. By default the retention of metrics stored by prometheus is set to 15d. If user wants to override this setting, this can be achieved by setting the `telemetry_prometheus_retention` value to desired duration.

#### Prometheus and KubeVirt

Prometheus is configured to scrape metrics related to VMs operated by KubeVirt by default.
#### 21.09 and the availability of Prometheus Dashboard

Users coming from Openness environment might be surprised by the lack of Prometheus Dashboard http endpoint running on port 3000. Due to security constraints for 21.09, the NodePort access to Prometheus dashboard is disabled. User concerned with this change can re-enable the NodePort (details for that will be presented in "How To" section) or use [port-forwarding feature](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/), forwarding port 80 of prometheus-server service.

### Grafana

Grafana is an open-source visualization and analytics software. It takes the data provided from external sources and displays relevant data to the user via dashboards. It enables the user to create customized dashboards based on the information the user wants to monitor and allows for the provision of additional data sources. In Intel® Smart Edge Open, the Grafana pod is deployed as a K8s Deployment type and is by default provisioned with data from Prometheus. It is enabled by default in DEK and can be enabled/disabled by changing the `telemetry_grafana_enable` flag (in provisioning config file). The detailed usage of Grafana will be presented in the "How To" section.

Default account is created with username `admin` and random password. Method of obtaining this password is shown in "How to" section.


## How To

### Log into Grafana

Requirements:

- Access to the node configured for `kubectl` usage with Intel® Smart Edge Open cluster.
- Network connection to EdgeNode
- One of [Grafana-supported browsers](https://grafana.com/docs/grafana/latest/installation/requirements/#supported-web-browsers)

Steps:

- Execute following command to obtain the password:
    ```[bash]
    kubectl get secrets/grafana -n telemetry -o json | jq -r '.data."admin-password"' | base64 -d
    ```
- Open `https://<nodeIP>:3200` in the browser
- Enter login and obtained password
  ![Grafana login](images/grafana_login_page.png)


### Check the collected of metrics in Prometheus using Grafana

Requirements:

- Network connection to EdgeNode
- Obtained login informations

Steps:

- Log in to Grafana using browser
- Select the "Explore" panel from the side-menu
  ![Explore option](images/grafana_explore_option.png)
- Enter the metric of interests (in this example it's `collectd_memory`) and press `Run Query`
  ![Result of executed query](images/grafana_explore.png)

### Create Grafana dashboard

Requirements:

- Network connection to EdgeNode
- Obtained login informations

Steps:

- Log into Grafana
- Go to "Dashboard panel" -> Manage. Alternatively user can click `Create` panel (big plus sign) and select `Dashboard` from the panel menu.
  ![Dashboard managment view](images/dashboards_manage.png)
- Click on `New Dashboard`
- In the following steps we will create simple dashboard showing memory and cpu usage on the node
  - Click `Add an empty panel`. First we're gonna add a Table showing cpu usage by container (cAdvisor must be running)
    - Enter `container_cpu_usage_seconds_total` in the `Metric browser` field. After selection grafana should ask to use the `rate` function. Do it and change the resolution to 1m.
    - Add title to the panel. For this example the panel is named `Container CPU usage`.
    - Change the panel resolution to `Last 5 minutes`
    ![Filled out panel info for cpu](images/grafana_add_panel_cpu.png)
    - Click `Apply`
  - Let's repeat the steps above for memory usage, this time the name of namespace is `Container memory usage`, metric is `container_memory_usage_bytes`. Set the rate function to 1m.
    - Set the unit to `bytes(IEC)`
    ![Filled out panel info for memory](images/grafana_add_panel_memory.png)
- Click `Save dashboard` and fill out the name. We used `Simple resource consumption stats`. Click `Save`

Comment:

This "How to" is non-exhaustive there is a plethora of available metrics and panels to chose from. For further we recommend visiting [official documentation for Grafana](https://grafana.com/docs/grafana/latest/).

### Enable prometheus Dashboard http NodePort

Requirements:

- Access to the node configured for `kubectl` usage with Intel® Smart Edge Open cluster.

Steps:

- Connect to the node
- Execute following command to expose the http dashboard endpoint on port 30000:

    ```[bash]
    kubectl patch svc -n telemetry prometheus-server --type='json' -p '[{"op":"replace","path":"/spec/type","value":"NodePort"},{"op":"replace","path":"/spec/ports/0/nodePort","value":30000}]'
    ```


### Instrument application to write StatsD metrics

For this "How to" a listing of simple Python application will be presented. For information how to use it with the language of your choosing please refer to language specific documentation:

Listing:
```[Python]
import random
import statsd
import time
from datetime import datetime

HOST = 'statsd-exporter.telemetry.svc' #This is the default statsd-exporter address in DEK
PORT = 8125

client = statsd.StatsClient(HOST, PORT)
start = time.time()

random.seed()
while True:
    random_sleep_interval = random.randint(5, 15)
    print('Got random interval to sleep: {}'.format(random_sleep_interval))

    time.sleep(random_sleep_interval)
    client.incr('statsd_script.sleep_calls')
    dt = int((time.time() - start) * 1000)
    client.timing('example.timer', dt)

    client.incr('example.counter', random_sleep_interval)
```
