### 📄 System Monitoring Stack Setup – Technical Summary

This document outlines the steps I followed to set up a system monitoring stack using **Prometheus**, **Node Exporter**, and **Grafana** to track CPU usage, memory usage, disk usage, and application uptime.

---

### ✅ Tasks Completed

1. **Setup of Prometheus & Node Exporter using Docker Compose**

   * Created a Docker Compose file to define and run Prometheus and Node Exporter containers.
   * Exposed the relevant ports to allow Prometheus to scrape metrics and for external access to dashboards.
   * Ensured Node Exporter was configured to collect metrics from the host system.

2. **Prometheus Configuration**

   * Defined scrape intervals and targets in a `prometheus.yml` configuration file.
   * Added Node Exporter as a scrape target so Prometheus can collect system-level metrics.

3. **Launching the Stack**

   * Brought up the Prometheus and Node Exporter containers using Docker Compose.
   * Verified Prometheus was successfully scraping metrics from Node Exporter.

4. **Grafana Installation and Setup**

   * Deployed Grafana either via Docker or local installation.
   * Accessed Grafana through the web UI and configured it for monitoring.

5. **Adding Prometheus as a Data Source in Grafana**

   * Configured a Prometheus data source in Grafana, pointing to the Prometheus service.
   * Verified the connection between Grafana and Prometheus.

6. **Importing Prebuilt Grafana Dashboard**

   * Imported a community-provided Grafana dashboard using its ID (e.g., for Node Exporter metrics).
   * Mapped it to the Prometheus data source and verified the charts displayed correctly.

---

### 🔍 Metrics Visualized

* CPU Usage
* Memory Usage
* Disk Usage
* Node/System Uptime

---

Let me know if you want this exported as a PDF or saved into a `.md` file!
