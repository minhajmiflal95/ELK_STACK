Centralized Log Analysis and Alerting System with the ELK Stack

This repository contains a step-by-step guide for deploying a robust, end-to-end log management and alerting system using the ELK (Elasticsearch, Logstash, Kibana) Stack on Ubuntu 22.04. This setup is ideal for monitoring infrastructure health, detecting security anomalies, and troubleshooting application issues in real-time.
A sample Kibana dashboard visualizing log data.
Architecture Overview
The system uses a standard, powerful architecture for log collection, processing, and visualization:
Filebeat → Logstash → Elasticsearch ← Kibana
Filebeat: A lightweight shipper that tails log files on your servers and forwards the data.
Logstash: A powerful server-side pipeline that processes and enriches the log data.
Elasticsearch: A distributed search and analytics engine that securely stores and indexes the logs for fast retrieval.
Kibana: A web interface for creating dashboards, exploring log data, and setting up alerts.
Phase 1: Installation and Setup
This setup assumes you are working on an Ubuntu 22.04 server with sudo privileges.
Step 1: Install Java
Elasticsearch requires the Java runtime.
sudo apt update
sudo apt install openjdk-17-jre -y

Step 2: Add the Elastic APT Repository
Import the Elasticsearch GPG Key:
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg


Add the Repository:
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list


Step 3: Install and Configure Core Components
Install Elasticsearch:
sudo apt update
sudo apt install elasticsearch -y
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch


Install Kibana:
sudo apt install kibana -y


To make the Kibana dashboard accessible from your network, edit its configuration file:
sudo nano /etc/kibana/kibana.yml


Uncomment and modify the following lines:
server.port: 5601
server.host: "0.0.0.0"


Then start and enable the service:
sudo systemctl start kibana
sudo systemctl enable kibana


You can access Kibana at http://<your_server_ip>:5601.
Phase 2: Log Collection and Processing
Step 1: Configure Filebeat
Install Filebeat on all servers from which you want to collect logs.
Install Filebeat:
sudo apt install filebeat -y


Enable the System Module: This module is pre-configured to collect system logs (syslog, auth logs).
sudo filebeat modules enable system


Configure Output to Logstash: Edit /etc/filebeat/filebeat.yml to send logs to your Logstash server.
# Comment out the elasticsearch output
# output.elasticsearch:
  # ...

# Uncomment and configure the logstash output
output.logstash:
  hosts: ["localhost:5044"] # Replace localhost with your Logstash server IP


Start Filebeat:
sudo systemctl start filebeat
sudo systemctl enable filebeat


Step 2: Configure Logstash
Install Logstash:
sudo apt install logstash -y


Create the Data Pipeline: Create a new configuration file to define the pipeline.
sudo nano /etc/logstash/conf.d/02-beats-input.conf


Add the following configuration to listen for Filebeat and send data to Elasticsearch:
input {
  beats {
    port => 5044
  }
}

# Filters can be added here to parse and transform logs
# filter { ... }

output {
  elasticsearch {
    hosts => ["http://localhost:9200"] # Replace localhost with your Elasticsearch IP
    index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+YYYY.MM.dd}"
  }
}


Start Logstash:
sudo systemctl start logstash
sudo systemctl enable logstash


Phase 3: Visualization and Alerting in Kibana
Step 1: Create an Index Pattern
In your Kibana dashboard, navigate to Stack Management > Index Patterns.
Click Create index pattern.
Define the pattern as filebeat-* and select @timestamp as the time filter field.
Step 2: Explore Logs
Navigate to the Discover tab to see your logs. You can use the Kibana Query Language (KQL) to search and filter your data.
Step 3: Set Up Alerts
Kibana can actively monitor your data and trigger alerts based on defined rules.
Example: Alert on Multiple Failed SSH Logins
Go to Stack Management > Rules and Connectors.
(Optional) Create a Connector to define a notification channel like Email or Slack.
Go to the Rules tab and click Create rule.
Rule type: Log threshold
Condition: Count where @timestamp is within the last 5 minutes.
Filter: system.auth.ssh.event : "Failed"
Threshold: is above 5
Configure an Action to use a connector and save the rule.
Your system will now alert you if more than five failed SSH login attempts are detected within a five-minute window.
