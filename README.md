# golang-loadblancer-keepalived
A simple HA config setup  with keepalived (VRRP) and golang load balancer

## Building

### Build for arn architecture

GOOS=linux GOARCH=arm go build main.go server.go -o golang-lb


### Build for amd architecture 

GOOS=linux GOARCH=amd64 go build main.go server.go -o golang-lb

### Config

Update the keepalived.conf file (change as required)

Copy the config file to /etc/keepalived/ to each of the load balancer servers (master and backup)

Start the keepalived service on both master and backup servers

```bash
sudo systemctl start keepalived
```

### Metrics

Install prometheus server and grafana on the server that will be used for metrics

Configure the prometheus yaml file - here is an example

```bash
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["192.168.8.161:9000","192.168.8.153:9000","192.168.8.156:9000","192.168.8.159:9000"]
```

Start Prometheus

```bash
sudo ./prometheus --config.file prometheus.yml
```

Start Grafana

```bash
sudo systemctl start grafana-server
```

In the Grafana webconsole load and change the file  `Pi Cluster [Whitebox Metrics]-1659885444442.json` as required.

## Usage

./golang-lb --backends=http://192.168.8.161:9000,http://192.168.8.153:9000,http://192.168.8.156:9000,http://192.168.8.159:9000


## Diagram

![Overview](keepalived-golang-lb.jpg)

## References

This project is a clone from https://github.com/kasvith/simplelb

I have added the config for HA keepalived to iot backend microservices

