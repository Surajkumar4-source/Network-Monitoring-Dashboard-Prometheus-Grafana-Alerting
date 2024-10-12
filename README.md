# Network Monitoring Stack - Prometheus, Grafana, and AlertManager

This project demonstrates a comprehensive network monitoring stack using **Prometheus**, **Grafana**, and **AlertManager** to collect, visualize, and manage alerts for system metrics.

## Overview

# Key Components of the Network Monitoring Stack

## 1. Prometheus

### Functionality:
Prometheus is a powerful open-source monitoring and alerting toolkit that focuses on reliability and scalability. It uses a **pull model** to scrape metrics from configured endpoints at specified intervals. Prometheus supports **multi-dimensional data collection** and querying.

### Data Storage:
Prometheus stores **time-series data** in a time-series database, allowing for easy querying with its powerful query language, **PromQL**. This enables users to analyze trends over time, correlate metrics, and set up alerts based on specific thresholds.

## 2. Grafana

### Purpose:
Grafana is an advanced **visualization and analytics platform** that integrates seamlessly with various data sources, including Prometheus. It enables users to create **customizable dashboards** that visualize metrics and logs in real-time.

### Visualization:
Grafana provides various visualization options, including **graphs**, **charts**, **heatmaps**, and **tables**, allowing users to tailor their monitoring views to specific needs and preferences.

## 3. Node Exporter

### Role:
Node Exporter is a specialized **Prometheus exporter** that runs on client nodes. It exposes a range of hardware and OS metrics, such as **CPU usage**, **memory consumption**,


## 2. Prerequisites

Before you begin the installation, ensure that the following prerequisites are in place:

- **Operating System**: The setup can be done on **Linux** (e.g., Ubuntu, CentOS), **Windows**, or **Docker**. This guide uses **Linux (Ubuntu 20.04 or 22.04)** as an example.
- **Root or Sudo Access**: You’ll need **administrative privileges** to install packages and configure services.
- **Basic Networking Knowledge**: Make sure you know how to access your system via IP, as these tools will run on specific ports.





# Project Setup

### 1. Prometheus Setup

#### Step 1: Install Prometheus

Download and install Prometheus on your centralized monitoring server:

     ```bash
           wget https://github.com/prometheus/prometheus/releases/download/v2.32.0/prometheus-2.32.0.linux-amd64.tar.gz
            tar xvf prometheus-2.32.0.linux-amd64.tar.gz
            cd prometheus-2.32.0.linux-amd64


Step 2: Configure Prometheus
Edit the prometheus.yml file to scrape metrics from client nodes:


     ```bash
         scrape_configs:
          - job_name: 'node_exporter'
            scrape_interval: 15s
            static_configs:
              - targets: ['<client_node_1>:9100', '<client_node_2>:9100'].  

##### Replace <client_node_1> and <client_node_2> with the actual IPs of your client nodes.

Step 3: Start Prometheus

     ```bash
          ./prometheus --config.file=prometheus.yml

### 2. Node Exporter Setup (Client Nodes)
#####  Install Node Exporter on each client node to expose system metrics:

        wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
        tar xvf node_exporter-1.3.1.linux-amd64.tar.gz
        cd node_exporter-1.3.1.linux-amd64
        ./node_exporter
### 3. Grafana Setup (Visualization)

##### Step 1: Install Grafana
##### Download and install Grafana:

     sudo apt-get install -y grafana
     sudo systemctl start grafana-server
##### Step 2: Configure Grafana

Access Grafana: http://<server_IP>:3000

Default login:

Username: admin

Password: admin

Add Prometheus as a data source:

Go to Settings → Data Sources

Select Prometheus and use http://localhost:9090 as the URL.

Create dashboards to visualize metrics like CPU, memory, and disk usage.

### 4. AlertManager Setup (Alerts)
Step 1: Install AlertManager
Download and install AlertManager:

    wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
    tar xvf alertmanager-0.23.0.linux-amd64.tar.gz
    cd alertmanager-0.23.0.linux-amd64

Step 2: Configure AlertManager
Edit the alertmanager.yml configuration file to set up alert receivers (email, Slack, etc.).

Step 3: Integrate with Prometheus

Modify prometheus.yml to include AlertManager:

    alerting:
      alertmanagers:
        - static_configs:
          - targets: ['localhost:9093']


Step 4: Define Alert Rules
Create alert rules in alert.rules.yml:

    groups:
      - name: alert.rules
        rules:
          - alert: HighCPUUsage
            expr: node_cpu_seconds_total{mode="idle"} < 10
            for: 1m
            labels:
              severity: critical
            annotations:
              summary: "High CPU Usage"
              description: "CPU usage is above threshold."

Load the rules into Prometheus:

      ./prometheus --config.file=prometheus.yml --web.enable-lifecycle


5.Generate dummy load using stress-ng package:

      stress-ng --cpu 4 --timeout 60s --metrics-brief



### How to Run

1.Start Prometheus:

    ./prometheus --config.file=prometheus.yml

2.Start Node Exporter on each client node: 
   
      ./node_exporter
      
3.Start Grafana:

     sudo systemctl start grafana-server

4.Start AlertManager:   

    ./alertmanager --config.file=alertmanager.yml

### Access
Prometheus: http://<server_IP>:9090

Grafana: http://<server_IP>:3000

AlertManager: http://<server_IP>:9093


### Conclusion

This setup provides a full-stack network monitoring solution using Prometheus for metrics collection, Grafana for visualization, and AlertManager for alerting.
      
 


            
            
            
          




     

              



