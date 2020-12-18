# Simplistic setup of fabric performance metrics capture

## Docker Compose changes

```
      - CORE_OPERATIONS_LISTENADDRESS=0.0.0.0:9090
      - CORE_METRICS_PROVIDER=prometheus
```

## bring up a prometheus server to retrieve and store the data

image name: prom/prometheus

args:         "Path": "/bin/prometheus",
        "Args": [
            "--config.file=/etc/prometheus/prometheus.yml",
            "--storage.tsdb.path=/prometheus",
            "--web.console.libraries=/usr/share/prometheus/console_libraries",
            "--web.console.templates=/usr/share/prometheus/consoles",
            "--web.enable-lifecycle"

config file
```
global:
  scrape_interval: 1s
  external_labels:
    monitor: 'devopsage-monitor'

scrape_configs:
  - job_name: prometheus
    honor_labels: true
    static_configs:
    - targets: ['localhost:7090', 'localhost:9090']
  - job_name: cadvisor
    scrape_interval: 5s
    static_configs:
    - targets: ['viper.cadvisor:8080']
```

```
docker run -v ~/caliper-workspace/prometheus:/etc/prometheus --name=prometheus --network=host prom/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/prometheus --web.console.libraries=/usr/share/promet
heus/console_libraries --web.console.templates=/usr/share/prometheus/consoles --web.enable-lifecycle --web.listen-address=:5090
```

# Setup graphna to visualize prometheus

image name: grafana/grafana

volume mappings
grafana_storage:/var/lib/grafana
/grafana/provisioning/:/etc/grafana/provisioning/
/var/run/docker.sock:/var/run/docker.sock

-e "GF_SECURITY_ADMIN_PASSWORD=admin"
-e "GF_USERS_ALLOW_SIGN_UP="false"

docker run -v ${PWD}/provisioning:/etc/grafana/provisioning --network=host --name graphana grafana/grafana

## config files
it’s possible to manage data sources in Grafana by adding one or more YAML config files in the provisioning/datasources directory

### datasource.yml
```
# config file version
apiVersion: 1

# list of datasources that should be deleted from the database
deleteDatasources:
  - name: Prometheus
    orgId: 1

# list of datasources to insert/update depending
# whats available in the database
datasources:
  # <string, required> name of the datasource. Required
- name: DS_PROMETHEUS
  # <string, required> datasource type. Required
  type: prometheus
  # <string, required> access mode. direct or proxy. Required
  access: proxy
  # <int> org id. will default to orgId 1 if not specified
  orgId: 1
  # <string> url
  url: http://viper.prometheus:9090
  # <string> database password, if used
  password:
  # <string> database user, if used
  user:
  # <string> database name, if used
  database:
  # <bool> enable/disable basic auth
  basicAuth: true
  # <string> basic auth username
  basicAuthUser: admin
  # <string> basic auth password
  basicAuthPassword: foobar
  # <bool> enable/disable with credentials headers
  withCredentials:
  # <bool> mark as default datasource. Max one per org
  isDefault: true
  # <map> fields that will be converted to json and stored in json_data
  jsonData:
     graphiteVersion: "1.1"
     tlsAuth: false
     tlsAuthWithCACert: false
  # <string> json object of data that will be encrypted.
  secureJsonData:
    tlsCACert: "..."
    tlsClientCert: "..."
    tlsClientKey: "..."
  version: 1
  # <bool> allow users to edit datasources from the UI.
  editable: true
  ```

### rebuilt dashboard
take fabric-1.4.json and import it

