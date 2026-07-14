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
