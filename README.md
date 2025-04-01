# Network Monitoring Stack - Prometheus, Grafana, and AlertManager

This project demonstrates a comprehensive network monitoring stack using **Prometheus**, **Grafana**, and **AlertManager** to collect, visualize, and manage alerts for system metrics.

# Centralized Monitoring with Prometheus, Grafana, and AlertManager

## Overview

A **centralized monitoring system** is essential for system administrators and DevOps teams to effectively manage, analyze, and visualize metrics from multiple servers, applications, or client nodes. This system consolidates data collection from various sources into a single platform, providing insights that facilitate proactive system management and troubleshooting.

## Key Components

### 1. Prometheus
Prometheus is an open-source monitoring and alerting toolkit widely used for gathering metrics from various endpoints. It operates on a **pull model**, scraping metrics from configured targets at regular intervals. Prometheus stores these metrics in a **time-series database**, making it easy to query and analyze data over time.

### 2. Grafana
Grafana is a powerful visualization and analytics tool that integrates with Prometheus and other data sources. It allows you to create customizable dashboards and visual representations of the data collected by Prometheus. Grafana enables teams to gain better insights into system performance and health.

### 3. Node Exporter
Node Exporter is a Prometheus exporter designed to expose hardware and OS metrics from client nodes. By running Node Exporter on each client machine, Prometheus can scrape metrics such as CPU usage, memory consumption, disk I/O, and network statistics, centralizing them for analysis and visualization.

### 4. AlertManager
**AlertManager** is an essential component of Prometheus that handles alerts generated by Prometheus when certain conditions are met. AlertManager manages the routing and delivery of these alerts to various destinations such as email, Slack, or PagerDuty. It allows you to define **alerting rules** in Prometheus, ensuring that administrators are notified when system issues arise or thresholds are breached.

#### Features:
- **Alert Routing**: Define custom routes for different alerts based on severity or other labels.
- **Alert Silencing**: Temporarily mute alerts during maintenance or known downtime.
- **Grouping**: Combine similar alerts into a single notification to reduce alert fatigue.

## Centralized Monitoring System Setup

In a centralized monitoring setup, we install **Prometheus** on a centralized server (master node) to monitor multiple client nodes. The client nodes run **Node Exporter** to expose their system metrics, which Prometheus scrapes and stores for visualization in Grafana. **AlertManager** ensures administrators are notified of any critical issues in real-time.

### Workflow:

1. Install and configure **Prometheus** on the centralized monitoring server.
2. Install and configure **Node Exporter** on each client node to expose metrics.
3. Set up **Grafana** for visualization by adding Prometheus as a data source and creating dashboards to view system metrics.
4. Configure **AlertManager** to handle alerts triggered by Prometheus when specified conditions are met.

### Benefits:
- **Unified Monitoring**: Monitor all your nodes from a single, centralized platform.
- **Real-time Metrics**: View real-time system metrics such as CPU, memory, disk I/O, and network usage.
- **Custom Dashboards**: Create visual dashboards in Grafana for a better understanding of your system's performance.
- **Proactive Alerting**: Ensure timely notification of system failures or performance issues through AlertManager.

## Installation Guide





## 2. Prerequisites

Before you begin the installation, ensure that the following prerequisites are in place:

- **Operating System**: The setup can be done on **Linux** (e.g., Ubuntu, CentOS), **Windows**, or **Docker**. This guide uses **Linux (Ubuntu 20.04 or 22.04)** as an example.
- **Root or Sudo Access**: You’ll need **administrative privileges** to install packages and configure services.
- **Basic Networking Knowledge**: Make sure you know how to access your system via IP, as these tools will run on specific ports.



<br>            
          
<br>

# Project Setup Implementation 


# Prometheus, Grafana, and Node Exporter Setup

## Step 1: Update and Install Dependencies
```bash
sudo apt update -y
sudo apt install -y wget unzip
```

## Step 2: Install Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.42.0/prometheus-2.42.0.linux-amd64.tar.gz
tar xvf prometheus-2.42.0.linux-amd64.tar.gz
cd prometheus-2.42.0.linux-amd64

sudo mv prometheus /usr/local/bin/
sudo mv promtool /usr/local/bin/
```

### Configure Prometheus
```bash
sudo mkdir -p /etc/prometheus
sudo touch /etc/prometheus/prometheus.yml
sudo nano /etc/prometheus/prometheus.yml
```
**Add the following content:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Create Prometheus Systemd Service
```bash
sudo nano /etc/systemd/system/prometheus.service
```
**Add the following content:**
```ini
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=root
Group=root
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/data \
  --web.listen-address=0.0.0.0:9090

[Install]
WantedBy=multi-user.target
```

### Start Prometheus Service
```bash
sudo mkdir -p /var/lib/prometheus/data
sudo chown -R root:root /var/lib/prometheus

sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

## Step 3: Install Node Exporter   (on al  clients)
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvf node_exporter-1.5.0.linux-amd64.tar.gz
sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
```

### Create Node Exporter Systemd Service
```bash
sudo nano /etc/systemd/system/node_exporter.service
```
**Add the following content:**
```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=root
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Start Node Exporter Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

## Step 4: Install Grafana
```bash
sudo apt-get install -y adduser libfontconfig1 musl
wget https://dl.grafana.com/oss/release/grafana_11.2.1_amd64.deb
sudo dpkg -i grafana_11.2.1_amd64.deb
```

### Start Grafana Service
```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```
**Dashboard Import ID:** `1860`

## Step 5: Install Alertmanager
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.25.0/alertmanager-0.25.0.linux-amd64.tar.gz
tar -xvzf alertmanager-0.25.0.linux-amd64.tar.gz
mv alertmanager-0.25.0.linux-amd64 alertmanager

sudo mv alertmanager/alertmanager /usr/local/bin/
sudo mv alertmanager/amtool /usr/local/bin/
```

### Configure Alertmanager
```bash
sudo mkdir -p /etc/alertmanager
sudo nano /etc/alertmanager/alertmanager.yml
```
**Add the following content:**
```yaml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: email_alert

receivers:
- name: email_alert
  email_configs:
  - to: csemanit2015@gmail.com
    from: aryan.arunachalam@gmail.com
    smarthost: smtp.gmail.com:587
    auth_username: aryan.arunachalam@gmail.com
    auth_identity: aryan.arunachalam@gmail.com
    auth_password: qzqyqmbeymmjtbbk
```

### Create Alertmanager Systemd Service
```bash
sudo nano /etc/systemd/system/alertmanager.service
```
**Add the following content:**
```ini
[Unit]
Description=Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=arun
ExecStart=/usr/local/bin/alertmanager --config.file=/etc/alertmanager/alertmanager.yml --storage.path=/var/lib/alertmanager/data
Restart=always

[Install]
WantedBy=multi-user.target
```

### Start Alertmanager Service
```bash
sudo mkdir -p /var/lib/alertmanager/data
sudo chown arun:arun /var/lib/alertmanager/data

sudo systemctl daemon-reload
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
```

## Step 6: Configure Prometheus for Alerting
```bash
sudo nano /etc/prometheus/prometheus.yml
```
**Add the following alerting configuration:**
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - 'localhost:9093'

rule_files:
  - "/etc/prometheus/alert.rules.yml"
```

### Create Alerting Rules
```bash
sudo touch /etc/prometheus/alert.rules.yml
sudo nano /etc/prometheus/alert.rules.yml
```
**Add the following content:**
```yaml
groups:
  - name: alert.rules
    rules:
      - alert: HighCPUUsage
        expr: 100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100) > 80
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "High CPU Usage"
          description: "CPU usage is above 80%."

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 80
        for: 1m
        labels:
          severity: warning
        annotations:
          summary: "High Memory Usage"
          description: "Memory usage is above 80%."

      - alert: NodeDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Node Down"
          description: "The node is not responding for more than 2 minutes."
```

### Restart Prometheus
```bash
sudo systemctl restart prometheus
```


### Step 7: Configure Grafana Dashboard

```yml

Open Grafana (http://localhost:3000)

Login with default credentials (admin/admin)

Add a Prometheus data source:

Navigate to Configuration > Data Sources

Select Prometheus

Set the URL to http://localhost:9090

Click Save & Test

Import a Grafana dashboard:

Go to Dashboards > Import

Enter Dashboard ID (e.g., 1860 for a Node Exporter dashboard)

Select Prometheus as the data source

```




## Setup Complete! 🎉



            
            
<br>            
          
<br>


### Screenshots

Below is a screenshot demonstrating the working of the project:

 #### 1. --------------------------------------------Grafana Running on Terminal----------------------------------
 
![Grafana](/screenshorts/grafana_terminal.png)

 #### 2. ---------------------------------------------promethises Running on Terminal-----------------------------
 
![Grafana](/screenshorts/promethises_terminal.png)


 #### 3. ---------------------------------------------Alertmanager Running on Terminal------------------------------
 
![Grafana](/screenshorts/alertmanager_terminal.png)



 #### 4. --------------------------------------------Node_exporter Running on Terminal----------------------------
 
![Grafana](/screenshorts/node_exporter_terminal.png)




 #### 5. ---------------------------------------------Grafana Running on localhost-------------------------------
 
![Grafana](/screenshorts/grafana___terminal.png)




 #### 6. -------------------------------------------Node_exporter Running on localhost------------------------------------
 
![Grafana](/screenshorts/node_exporter.png)


 #### 7. -------------------------------------------promethises Running on localhost----------------------------------------
 
![Grafana](/screenshorts/rules.png)


 #### 8. ---------------------------------------------Alertmanager Running on localhost------------------------------------
 
![Grafana](/screenshorts/alertmanager.png)
     

              
 #### 9. -------------------------------------------Dummy_load running Terminal---------------------------------------------
 
![Grafana](/screenshorts/dummy_load.png)


 #### 10. ---------------------------------------Targets Running on localhost-----------------------------------------------
 
![Grafana](/screenshorts/targets.png)






## Working Demo Video

https://github.com/user-attachments/assets/cb40c7bf-3fcd-475d-a0d3-bd12c5c21d17











## Summary of the Project: Centralized Monitoring with Prometheus, Grafana, and AlertManager

After completing the configuration of Prometheus, Grafana, Node Exporter, and AlertManager, the centralized monitoring system is fully set up. This system allows you to:

Monitor Multiple Nodes: Prometheus scrapes metrics from the client nodes running Node Exporter, centralizing hardware and OS data such as CPU, memory, disk I/O, and network usage.

Visualize Metrics: Grafana provides customizable dashboards where you can visualize the metrics collected by Prometheus. This enables you to get a real-time view of the performance and health of all client nodes in a single interface.

Set Up Alerts: With AlertManager, you can configure alerts based on defined thresholds. For example, an alert can be triggered if CPU utilization on any client node exceeds 50%. When this happens, the system automatically sends an alert via email or other channels configured in AlertManager.

## Proactive System Management: This monitoring solution allows system administrators and DevOps teams to respond quickly to any issues by getting notified in real-time when system thresholds are breached. This helps in maintaining system reliability, reducing downtime, and optimizing resource usage.


This dashboard provides a clear, visual overview of all your monitored systems, allowing you to track system performance and respond proactively to any potential issues.

With this setup, you can ensure that your infrastructure is continuously monitored, and alerts are triggered for quick resolution of critical issues, such as CPU overutilization or system health degradation.

Below is an screnshort , we get when alert occurs by alertmanager:




![mail  blur](https://github.com/user-attachments/assets/35ddacc1-9b62-4b0a-a7d8-3b9ec6d8fdfc)















<br>
<br>
<br>
<br>



**👨‍💻 𝓒𝓻𝓪𝓯𝓽𝓮𝓭 𝓫𝔂**: [Suraj Kumar Choudhary](https://github.com/Surajkumar4-source) | 📩 **𝓕𝓮𝓮𝓵 𝓯𝓻𝓮𝓮 𝓽𝓸 𝓓𝓜 𝓯𝓸𝓻 𝓪𝓷𝔂 𝓱𝓮𝓵𝓹**: [csuraj982@gmail.com](mailto:csuraj982@gmail.com)





<br>
