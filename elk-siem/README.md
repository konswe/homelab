# ELK Stack SIEM Deployment

## Project Overview
Deployment of a centralized Security Information and Event Management (SIEM) system using the Elastic Stack (Elasticsearch, Logstash, Kibana). The goal of this project is to collect, parse, and visualize system logs for security monitoring and incident response.

## Prerequisites & Environment
*   **OS:** Ubuntu Server 26.04 LTS
*   **Resources:** 8GB RAM allocated to Elasticsearch JVM Heap (Host total: 64GB RAM)
*   **Core Components:** Elasticsearch (v8.x), Logstash (v8.x), Kibana (v8.x)

## Architecture
*   **Elasticsearch:** Core database, search and analytics engine (Port 9200).
*   **Logstash:** Server-side data processing pipeline to ingest and parse logs (Port 5044).
*   **Kibana:** Web-based interface for data visualization and dashboards (Port 5601).
*   **Data Sources:** **(WIP)**

## Installation & Setup

### 1. Repository Setup & Elasticsearch Installation
Commands used to set up the official Elastic repository and install the core database:

**(WIP)**

```
sudo apt update && sudo apt upgrade -y
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin


mkdir -p ~/homelab/elk-siem/logstash/pipeline
cd ~/homelab/elk-siem
nano docker-compose.yml

```

```
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:9.4.3
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ports:
      - "9200:9200"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  kibana:
    image: docker.elastic.co/kibana/kibana:9.4.3
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  logstash:
    image: docker.elastic.co/logstash/logstash:9.4.3
    container_name: logstash
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5044:5044"
    environment:
      - xpack.monitoring.enabled=false
    depends_on:
      - elasticsearch
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  es_data:
    driver: local
```

```
nano logstash/pipeline/logstash.conf

input {
  beats {
    port => 5044
  }
}

filter {
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logstash-%{+YYYY.MM.dd}"
  }
}
```
Check Troubleshooting & Lessons Learned section
```
sudo docker compose up -d
```

```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-9.x.list
sudo apt-get update && sudo apt-get install filebeat
```

sudo nano /etc/filebeat/filebeat.yml
```
filebeat.inputs:
  enabled: false -> true

Comment this section
#output.elasticsearch:

uncomment this section
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5044"]
```


sudo filebeat modules enable system

The name of this file will be different if you dont enable the filebeat.

sudo nano /etc/filebeat/modules.d/system.yml

```

# Module: system
# Docs: https://www.elastic.co/guide/en/beats/filebeat/9.4/filebeat-module-system.html

- module: system
  # Syslog
  syslog:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

    # Use journald to collect system logs
    #var.use_journald: false

  # Authorization logs
  auth:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

    # Use journald to collect auth logs
    #var.use_journald: false

```

sudo systemctl enable filebeat
sudo systemctl start filebeat

<img width="1901" height="350" alt="image" src="https://github.com/user-attachments/assets/74c32ba8-bd74-463d-a7a1-d477e3c99afd" />


## Troubleshooting & Lessons Learned

* **Issue:** Encountered network timeouts and authorization errors during the initial `docker compose up -d` execution. The large size of the ELK stack images caused the connection to the Elastic Docker registry to drop, interrupting the pull process for Logstash and Kibana.
    * **Exact Errors Logged:**
        ```text
        Error response from daemon: failed to copy: httpReadSeeker: failed open: failed to do request: Get "[...]": dial tcp 34.56.16.77:443: i/o timeout
        ```
        ```text
        Error response from daemon: failed to resolve reference "docker.elastic.co/kibana/kibana:9.4.3": failed to authorize: failed to fetch anonymous token: Get "[...]": dial tcp 34.56.16.77:443: i/o timeout
        ```
* **Fix:** Simply re-ran the `docker compose up -d` command. Docker natively caches partially downloaded image layers, so consecutive executions resumed the download process from where it left off until all images were fully pulled and containers successfully started (status `[+] up 19/19`).
