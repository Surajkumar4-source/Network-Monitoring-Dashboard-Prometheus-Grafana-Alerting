# Network Monitoring Stack - Prometheus, Grafana, and AlertManager

This project demonstrates a comprehensive network monitoring stack using Prometheus, Grafana, and AlertManager to collect, visualize, and manage alerts for system metrics.

## Project Setup

```bash
# 1. Prometheus Setup

# Step 1: Install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.32.0/prometheus-2.32.0.linux-amd64.tar.gz
tar xvf prometheus-2.32.0.linux-amd64.tar.gz
cd prometheus-2.32.0.linux-amd64

# Step 2: Configure Prometheus (edit prometheus.yml)
cat <<EOT >> prometheus.yml
scrape_configs:
  - job_name: 'node_exporter'
    scrape_interval: 15s
    static_configs:
      - targets: ['<client_node_1>:9100', '<client_node_2>:9100']
EOT

# Replace <client_node_1> and <client_node_2> with actual IPs of your client nodes.

# Step 3: Start Prometheus
./prometheus --config.file=prometheus.yml

# 2. Node Exporter Setup (Client Nodes)

# Install Node Exporter on each client node
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvf node_exporter-1.3.1.linux-amd64.tar.gz
cd node_exporter-1.3.1.linux-amd64
./node_exporter

# Node Exporter will expose metrics at http://<client_IP>:9100/metrics

# 3. Grafana Setup (Visualization)

# Step 1: Install Grafana
sudo apt-get install -y grafana
sudo systemctl start grafana-server

# Step 2: Access Grafana at http://<server_IP>:3000
# Default login: Username: admin | Password: admin

# Add Prometheus as a data source
# Navigate to Settings → Data Sources → Select Prometheus → Use http://localhost:9090 as the URL
# Create dashboards to visualize metrics like CPU, memory, and disk usage

# 4. AlertManager Setup (Alerts)

# Step 1: Install AlertManager
wget https://github.com/prometheus/alertmanager/releases/download/v0.23.0/alertmanager-0.23.0.linux-amd64.tar.gz
tar xvf alertmanager-0.23.0.linux-amd64.tar.gz
cd alertmanager-0.23.0.linux-amd64

# Step 2: Configure AlertManager (edit alertmanager.yml)
cat <<EOT >> alertmanager.yml
receivers:
  - name: 'default-receiver'
    email_configs:
      - to: 'your_email@example.com'
EOT

# Step 3: Integrate AlertManager with Prometheus
# Add to prometheus.yml:
cat <<EOT >> prometheus.yml
alerting:
  alertmanagers:
    - static_configs:
      - targets: ['localhost:9093']
EOT

# Step 4: Define Alert Rules (create alert.rules.yml)
cat <<EOT >> alert.rules.yml
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
EOT

# Load the rules into Prometheus
./prometheus --config.file=prometheus.yml --web.enable-lifecycle

# How to Run

# Start Prometheus
./prometheus --config.file=prometheus.yml

# Start Node Exporter on each client node
./node_exporter

# Start Grafana
sudo systemctl start grafana-server

# Start AlertManager
./alertmanager --config.file=alertmanager.yml

# Access URLs:
# Prometheus: http://<server_IP>:9090
# Grafana: http://<server_IP>:3000
# AlertManager: http://<server_IP>:9093
