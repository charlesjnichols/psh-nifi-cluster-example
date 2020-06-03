create a docker network connect the nifi container
```
docker network create flowapp
```

create a new nifi container
```
docker run --name nifi -p 8080:8080 -p 9092:9092 --network=flowapp -d apache/nifi:latest
```
http://localhost:8080/nifi/
import the integration_test template

add prometheus reporting task

navigate to /metrics exposure


create a prometheus yml that harvests the metrics from nifi
```
>prometheus.yml

# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['127.0.0.1:9090']

  - job_name: 'nifi'
    metrics_path: '/metrics'
    scrape_interval: 5s
    static_configs:
    - targets: ['nifi:9092']
```

create a new prometheus container
```
docker run -d --name prometheus -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml --network=flowapp prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```

navigate to the job status page to verify nifi metrics are being harvested
http://localhost:9090/targets

nagivate to the graph query page to verify the Processor metrics are collected
http://localhost:9090/graph?g0.range_input=1h&g0.expr=nifi_amount_bytes_written%7Bcomponent_type%3D%22Processor%22%7D&g0.tab=1

create a new grafana container
```
docker run -d --name grafana -p 3000:3000 --network flowapp grafana/grafana
```

navigate to grafana and login using the default creds admin:admin
http://localhost:3000/login

add prometheus as a new datasource

save a test


create a new dashboard

add a panel

add the nifi metrics as a line graph

configure the legend to show the component_names


