### 📄 System Monitoring Stack Setup 

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

   * Deployed Grafana using Docker Compose alongside Prometheus.
   * Accessed Grafana through the web UI and completed initial setup.

5. **Adding Prometheus as a Data Source in Grafana**

   * Configured a Prometheus data source in Grafana, pointing to the Prometheus service.
   * Verified the connection between Grafana and Prometheus.

6. **Created Grafana Dashboard**

   * Created a Grafana dashboard.
   * Mapped it to the Prometheus data source and verified the charts displayed correctly.

---

### Configuring Grafana Alerting with Email (SMTP)

I set up **Grafana alerts** to send notifications via email when critical system thresholds are reached (e.g., CPU usage > 80%).

#### Steps:

1. **Configure SMTP in Grafana**

   Updated `grafana.ini` configuration (if self-hosted), or created a custom config for Docker with SMTP settings like this:

   ```ini
   [smtp]
   enabled = true
   host = smtp.mailtrap.io:587
   user = <SMTP_USERNAME>
   password = <SMTP_PASSWORD>
   from_address = alerts@example.com
   from_name = Grafana Alerts

   [emails]
   welcome_email_on_sign_up = false
   ```

   > For Docker Compose, mount this config into the Grafana container:

   ```yaml
   grafana:
     image: grafana/grafana
     volumes:
       - ./grafana.ini:/etc/grafana/grafana.ini
   ```

2. **Enable SMTP in Docker Compose Environment (Alternative)**

   If not using `grafana.ini`, environment variables can be added to the service:

   ```yaml
   environment:
     - GF_SMTP_ENABLED=true
     - GF_SMTP_HOST=smtp.mailtrap.io:587
     - GF_SMTP_USER=<SMTP_USERNAME>
     - GF_SMTP_PASSWORD=<SMTP_PASSWORD>
     - GF_SMTP_FROM_ADDRESS=alerts@example.com
     - GF_SMTP_FROM_NAME=Grafana Alerts
   ```

3. **Create an Alert Rule in Grafana**

   * Navigated to a dashboard panel (e.g., CPU load panel).
   * Clicked the **Alert** tab and configured a condition (e.g., "avg() of CPU load is above 80 for 5m").
   * Defined the alert frequency and evaluation time.

4. **Configure Notification Channel (Email)**

   * Went to **Alerting > Contact points** in Grafana.
   * Created a **New Contact Point** with type **Email**.
   * Added recipient address(es) and tested delivery.

5. **Attach Alert Rule to the Contact Point**

   * Under **Alert Rules**, connected the alert to the newly created email contact point.
   * Saved and tested the alerting configuration.

---

### 🔍 Metrics Visualized

* CPU Usage
* Memory Usage
* Disk Usage
* System Uptime

---

Grafana and Prometheus can also give insight into applications like Django.
