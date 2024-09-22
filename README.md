# Aggregator
```
#podman run -d --name vector-aggregator -v $(pwd)/aggregator:/etc/vector/ --rm -p 9090:9090 -p 8686:8686 -p 6000:6000 docker.io/timberio/vector:0.41.1-alpine
docker run -d --name vector-aggregator -v $(pwd)/aggregator:/etc/vector/ --rm -p 9090:9090 -p 8686:8686 -p 6000:6000 docker.io/timberio/vector:0.41.1-alpine
#docker rm -f vector-aggregator 
```
source: https://vector.dev/docs/reference/configuration/sources/docker_logs/
transform: https://vector.dev/docs/reference/configuration/transforms/ 
sink: https://vector.dev/docs/reference/configuration/sinks/vector/ 
metric: https://vector.dev/docs/about/under-the-hood/architecture/data-model/metric/

# Agent
```
# podman run -d --name vector-agent -v $(pwd)/agent:/etc/vector/ --rm docker.io/timberio/vector:0.41.1-alpine
docker run -d --name vector-agent -v $(pwd)/agent:/etc/vector/ --rm docker.io/timberio/vector:0.41.1-alpine
#docker rm -f vector-agent 
```
source: https://vector.dev/docs/reference/configuration/sources/docker_logs/
transform: https://vector.dev/docs/reference/configuration/transforms/ 
sink: https://vector.dev/docs/reference/configuration/sinks/vector/ 

#### aggregator/vector.yaml
```
mkdir -p aggregator
cat <<-EOF > $PWD/aggregator/vector.yaml
data_dir: /vector-data-dir
api:
  enabled: true
  address: 0.0.0.0:8686
sources:
  vector:
     address: 0.0.0.0:6000
     type: vector
     version: "2"
  internal_metrics:
      type: internal_metrics 
transforms:
  parse_logs:
    type: "remap"
    inputs: ["vector"]
    source: |
      . = parse_syslog!(string!(.message)) 
sinks:
  prom_exporter:
    type: prometheus_exporter
    inputs: [internal_metrics]
    address: 0.0.0.0:9090
  stdout:
    type: console
    inputs: [parse_logs]
    encoding:
        codec: json
EOF
```
# /agent/vector.yaml
```
mkdir -p agent
cat <<-EOF > $PWD/agent/vector.yaml
data_dir: /vector-data-dir
api:
  enabled: false
  address: 0.0.0.0:8686
sources:
  dummy_logs:
    type: "demo_logs"
    format: "syslog"
    interval: 1
sinks:
  vector_sink:
    type: vector
    inputs:
      - dummy_logs
    address: 172.17.0.1:6000         #Change me
  stdout:
    type: console
    inputs: [dummy_logs]
    encoding:
        codec: json
EOF
```




