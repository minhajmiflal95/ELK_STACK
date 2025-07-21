
# Centralized Log Analysis and Alerting System with the ELK Stack

A hands-on lab and complete guide to building a centralized log analysis and incident response platform using the **ELK Stack** (Elasticsearch, Logstash, Kibana) on **Ubuntu 22.04**. This setup is ideal for:

- Monitoring infrastructure health  
- Detecting security anomalies  
- Troubleshooting application issues in real-time  

---

## 🔧 Architecture Overview

```text
Filebeat → Logstash → Elasticsearch ← Kibana
```

- **Filebeat**: Lightweight shipper for forwarding logs
- **Logstash**: Pipeline for ingesting and transforming log data
- **Elasticsearch**: Search and analytics engine for storing and indexing logs
- **Kibana**: Visualization tool for dashboards and alerting

---

## 📦 Phase 1: Installation and Setup

### Step 1: Install Java
```bash
sudo apt update
sudo apt install openjdk-17-jre -y
```

### Step 2: Add the Elastic APT Repository
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

### Step 3: Install and Configure Core Components

#### Elasticsearch
```bash
sudo apt update
sudo apt install elasticsearch -y
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
```

#### Kibana
```bash
sudo apt install kibana -y
sudo nano /etc/kibana/kibana.yml
```

Uncomment and set:
```yaml
server.port: 5601
server.host: "0.0.0.0"
```

Start Kibana:
```bash
sudo systemctl start kibana
sudo systemctl enable kibana
```

Access: `http://<your_server_ip>:5601`

---

## 📥 Phase 2: Log Collection and Processing

### Step 1: Configure Filebeat

```bash
sudo apt install filebeat -y
sudo filebeat modules enable system
```

Edit `/etc/filebeat/filebeat.yml`:
```yaml
# Disable Elasticsearch output
# output.elasticsearch:

# Enable Logstash output
output.logstash:
  hosts: ["<logstash_server_ip>:5044"]
```

Start Filebeat:
```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat
```

### Step 2: Configure Logstash

```bash
sudo apt install logstash -y
sudo nano /etc/logstash/conf.d/02-beats-input.conf
```

Paste the following configuration:

```conf
input {
  beats {
    port => 5044
  }
}

# Add filters here if needed

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}
```

Start Logstash:
```bash
sudo systemctl start logstash
sudo systemctl enable logstash
```

---

## 📊 Phase 3: Visualization and Alerting in Kibana

### Step 1: Create an Index Pattern

1. Go to **Kibana > Stack Management > Index Patterns**
2. Click **Create Index Pattern**
3. Enter `filebeat-*` as pattern, use `@timestamp` as the time filter

### Step 2: Explore Logs

- Navigate to **Discover** tab
- Use **Kibana Query Language (KQL)** for filtering log data

### Step 3: Set Up Alerts (e.g. SSH brute-force detection)

1. Go to **Stack Management > Rules and Connectors**
2. (Optional) Create a connector for Email/Slack notifications
3. Click **Create Rule**
   - Rule Type: `Log threshold`
   - Condition: Count logs in the last 5 minutes
   - Filter: `system.auth.ssh.event : "Failed"`
   - Threshold: above `5`
4. Assign an action (e.g., email alert) and save the rule

---

## 📌 Summary

- ✅ Centralized, scalable log monitoring
- 🔍 Real-time log visualization and querying
- 📢 Alerting and incident detection

---

> For issues, improvements, or contributions, feel free to fork or open an issue.
