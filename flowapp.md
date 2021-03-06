## Network Setup

create a docker network to connect the containers
```
docker network create flowapp
```

## Nifi Setup

create a new nifi container
```
docker run --name nifi -p 8080:8080 -p 9092:9092 --network=flowapp -d apache/nifi:latest
```
navigate to the nifi graph http://localhost:8080/nifi/ and import the [flowapp_integration_test.xml](flowapp_integration_test.xml?raw=true) template
![snips](snips/apply_template.png?raw=true)

add prometheus reporting task to the controller settings
![snips](snips/reporting_task_add.png?raw=true)
![snips](snips/reporting_task_search.png?raw=true)
![snips](snips/reporting_task_config.png?raw=true)
![snips](snips/reporting_task_started.png?raw=true)

start the graph processors and navigate to prometheus metrics exposure http://localhost:9092/metrics/
![snips](snips/reporting_task_endpoint.png?raw=true)

## Prometheus Setup
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

create a new prometheus container mounting the configuration file
```
docker run -d --name prometheus -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml --network=flowapp prom/prometheus --config.file=/etc/prometheus/prometheus.yml
```

navigate to the job status page to verify nifi metrics are being harvested http://localhost:9090/targets
![snips](snips/prometheus_jobs.png?raw=true)

nagivate to the graph query page to verify the Processor metrics are collected http://localhost:9090/graph?g0.range_input=1h&g0.expr=nifi_amount_bytes_written%7Bcomponent_type%3D%22Processor%22%7D&g0.tab=1
![snips](snips/prometheus_query.png?raw=true)

## Grafana Setup
create a new grafana container
```
docker run -d --name grafana -p 3000:3000 --network flowapp grafana/grafana
```

navigate to grafana and login using the default creds admin:admin http://localhost:3000/login
![snips](snips/grafana_login.png?raw=true)

add prometheus as a new datasource
![snips](snips/grafana_datasource.png?raw=true)
![snips](snips/grafana_datasource_add.png?raw=true)
![snips](snips/grafana_datasource_config.png?raw=true)
![snips](snips/grafana_datasource_save.png?raw=true)


create a new dashboard
![snips](snips/grafana_daashboard_create.png?raw=true)

add a panel
![snips](snips/grafana_dashboard_create_panel.png?raw=true)

add the nifi metrics as a line graph
![snips](snips/grafana_panel_edit.png?raw=true)

configure the legend to show the component_names
![snips](snips/grafana_panel_legend.png?raw=true)

configure the panel settings
![snips](snips/grafana_panel_settings.png?raw=true)

and view the dashboard
![snips](snips/grafana_flowapp.png?raw=true)


