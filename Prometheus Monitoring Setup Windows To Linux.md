
# Overview
- Prometheus hub running on Windows
- Dns exporter running on Linux Docker containers
- Linux Docker containers are configured using home-grown "sacm" tool

# First Steps
1. Download Prometheus for Windows (portable)
2. Start Prometheus on Windows immediately to see it work: 
```bash=
.\prometheus.exe --config.file .\prometheus.yml
```
3. Go to `https:\\localhost:9090` in a browser and see the Prometheus page 

Setup Docker DNS Exporters for Unbound DNS 

- Unbound servers running as containers on Linux
- Unbound runs on Docker using `sacm` with this `INIT.cfg` file:

```bash=
DOCKER_NAME="unbound-rpi"
DOCKER_RUN_COMMAND="docker run \
         --name=unbound-rpi \
         --volume=${REPO_CFG_DIR}/a-records.conf:/opt/unbound/etc/unbound/a-records.conf:ro \
         --publish=53:53/udp \
         --publish=53:53/tcp \
         --restart=unless-stopped \
         --detach=true \
         mvance/unbound-rpi:latest"

```

Create an `INIT.cfg` file for the `dns_exporter`:
```bash=
DOCKER_NAME="dns_exporter"
DOCKER_RUN_COMMAND="docker run \
         --name=dns_exporter \
         --restart=unless-stopped \
         --detach=true \
         --publish 15353:15353 \
        tykling/dns_exporter:latest"
```

Update the `prometheus.yml` to get the data being published by `dns_exporter` and note all the lines with  `pi4`, the hostname, listed within them:

```yaml=
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
# The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ["localhost:9090"]
        # The label name is added as a label `label_name=<label_value>` to any timeseries scraped from this config.
        labels:
          app: "prometheus"
  - job_name: "dnsexp_pi4"
    scrape_interval: "10s"
    metrics_path: "/metrics"
    relabel_configs:
      - target_label: "monitor"
        replacement: "pi4:115353"
    static_configs:
      - targets:
        - "pi4:15353"
  - job_name: "dnsexp_pi4_rec"
    scrape_interval: "10s"
    metrics_path: "/query"
    params:
      module:
        - "quad9_mx"
    relabel_configs:
      - source_labels: ["__address__"]
        target_label: "__param_query_name"
      - source_labels: ["__address__"]
        target_label: "instance"
      - target_label: "__address__"
        replacement: "pi4:15353"
      - target_label: "monitor"
        replacement: "pi4:15353"
    static_configs:
      - targets:
          - "gmail.com"
          - "outlook.com"
```

Restart prometheus on Windows and go to http://localhost:9090 again to see the new data with the `dns_exporter`.

