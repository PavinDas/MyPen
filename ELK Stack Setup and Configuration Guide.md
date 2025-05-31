
#  ELK Stack Setup and Configuration Guide

  

This guide provides step-by-step instructions to set up the ELK Stack (Elasticsearch, Logstash, Kibana) for log management and analysis. It covers installation, configuration, data ingestion, log parsing, and visualization.

  ![enter image description here](https://miro.medium.com/v2/resize:fit:1400/1*vZDu4Bwj2GxQh8t1IjDq4w.png)

##  Prerequisites

- Operating System: Ubuntu 20.04 (or similar Linux distribution)

- Java: OpenJDK 11 or 17 (required for Elasticsearch and Logstash)

- Minimum Hardware: 4GB RAM, 2 CPU cores, 20GB disk space

- Network: Ports 9200 (Elasticsearch), 5044 (Logstash), 5601 (Kibana) open

  

##  Step 1: Install Elasticsearch

Elasticsearch stores and indexes logs for fast searching.

  

1.  **Install Java**

```bash

sudo apt update

sudo apt install openjdk-11-jdk -y

java -version

```

  

2.  **Add Elasticsearch Repository**

```bash

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch |  sudo  gpg  --dearmor  -o  /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main"  |  sudo  tee  /etc/apt/sources.list.d/elastic-8.x.list

```

  

3.  **Install Elasticsearch**

```bash

sudo apt update

sudo apt install elasticsearch -y

```

  

4.  **Configure Elasticsearch**

Edit `/etc/elasticsearch/elasticsearch.yml`:

```yaml

cluster.name:  elk-cluster

node.name:  node-1

network.host:  127.0.0.1

http.port:  9200

```

  

5.  **Start and Enable Elasticsearch**

```bash

sudo systemctl start elasticsearch

sudo systemctl enable elasticsearch

```

  

6.  **Verify Installation**

```bash

curl -X GET "http://localhost:9200"

```

Expected output: JSON response with cluster details.

  

##  Step 2: Install Logstash

Logstash processes and parses logs before sending them to Elasticsearch.

  

1.  **Install Logstash**

```bash

sudo apt install logstash -y

```

  

2.  **Configure Logstash**

Create a sample configuration file at `/etc/logstash/conf.d/logstash-sample.conf`:

```conf

input {

file {

path  => "/var/log/app.log"

start_position  => "beginning"

}

}

filter {

grok {

match  => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:loglevel} %{GREEDYDATA:message}" }

}

}

output {

elasticsearch {

hosts  => ["http://localhost:9200"]

index  => "logs-%{+YYYY.MM.dd}"

}

stdout { codec  => rubydebug }

}

```

  

3.  **Test Configuration**

```bash

sudo /usr/share/logstash/bin/logstash --config.test_and_exit -f /etc/logstash/conf.d/logstash-sample.conf

```

  

4.  **Start and Enable Logstash**

```bash

sudo systemctl start logstash

sudo systemctl enable logstash

```

  

##  Step 3: Install Kibana

Kibana provides a web interface for visualizing and analyzing logs.

  

1.  **Install Kibana**

```bash

sudo apt install kibana -y

```

  

2.  **Configure Kibana**

Edit `/etc/kibana/kibana.yml`:

```yaml

server.port:  5601

server.host:  "127.0.0.1"

elasticsearch.hosts: ["http://localhost:9200"]

```

  

3.  **Start and Enable Kibana**

```bash

sudo systemctl start kibana

sudo systemctl enable kibana

```

  

4.  **Access Kibana**

Open a browser and navigate to `http://localhost:5601`. Log in with default credentials (if security is enabled, configure users via Elasticsearch).

  

##  Step 4: Data Ingestion and Log Parsing

-  **Sample Log File**: Create `/var/log/app.log` with sample logs:

```

2025-05-31T13:45:00+05:30 INFO Application started successfully

2025-05-31T13:45:01+05:30 ERROR Failed to connect to database

```

-  **Logstash Processing**: Logstash reads `app.log`, parses it using the `grok` filter, and sends it to Elasticsearch.

-  **Index Creation**: Logs are stored in Elasticsearch under indices like `logs-2025.05.31`.

  

##  Step 5: Visualization in Kibana

1.  **Access Kibana**: Go to `http://localhost:5601`.

2.  **Create Index Pattern**:

- Navigate to **Management > Stack Management > Index Patterns**.

- Create a pattern: `logs-*`.

- Set `@timestamp` as the time field.

3.  **Create Visualizations**:

- Go to **Visualize > Create Visualization**.

- Choose a chart (e.g., Vertical Bar).

- Select index pattern `logs-*`.

- Example: Plot log levels (e.g., INFO, ERROR) over time.

- Y-axis: Count

- X-axis: `@timestamp` (bucket by hour)

4.  **Create Dashboard**:

- Go to **Dashboard > Create Dashboard**.

- Add your visualization.

- Save and view your log analysis dashboard.

  

##  Troubleshooting

-  **Elasticsearch not running**: Check status with `sudo systemctl status elasticsearch`.

-  **Logstash errors**: View logs at `/var/log/logstash/logstash-plain.log`.

-  **Kibana access issues**: Ensure `server.host` is correct and port 5601 is open.

  

##  Next Steps

- Enable security: Configure X-Pack security in Elasticsearch and Kibana.

- Scale: Add more nodes to Elasticsearch for a cluster setup.

- Ingest more data: Configure Logstash inputs for other sources (e.g., syslog, beats).
