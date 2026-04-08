# Factor Telemetry

Welcome to **Factor Telemetry**, a comprehensive repository of ready-to-use telemetry integrations and dashboard templates for monitoring Apache Kafka and Apache Flink environments. These configurations are designed to visualize the rich, Prometheus-compatible metrics emitted by [Factor House](https://factorhouse.io/) products, including Kpow and Flex.

While observability is often associated with Grafana, this project is built to be platform-agnostic. The metrics exposed by Kpow and Flex can be seamlessly integrated into a wide variety of modern monitoring and alerting platforms, including Grafana, Datadog, New Relic, and more. 

### 🗂️ Organization

Inside this repository, you will find platform-specific configuration files and templates. Currently, our Grafana JSON models are located within the `grafana-templates` directory. To keep things cleanly organized, these dashboards are further divided into dedicated subfolders based on the target product:

* The **`kpow`** folder contains all of our Kafka-focused dashboards (covering environments, topics, consumer groups, and Kafka Connect).
* The **`flex`** folder contains dashboards dedicated to Flink cluster monitoring. 

As we expand support for other platforms like Datadog, additional directories will be added to help you quickly deploy world-class observability into your tool of choice.

## 📊 Dashboards Overview

**Quality Gap of Raw JMX Data vs. High-Fidelity Metrics**

The standard approach of routing raw Kafka JMX metrics into Prometheus often leaves teams with noisy dashboards and fragile alerts. Attempting to compute meaningful business metrics, like exact consumer lag or active throughput, from raw JMX offsets using PromQL is notoriously difficult.

**These templates take a different approach.** They are built on top of Kpow, which acts as a high-fidelity metrics engine. Instead of relying on JMX sidecars, Kpow directly observes your cluster and exposes pre-calculated, actionable metrics (such as exact `group_offset_lag` and `topic_end_delta`) ready for immediate visualization.

📖 **Read the architectural deep-dive: **[Beyond JMX: Supercharging Grafana Dashboards with High-Fidelity Metrics](https://factorhouse.io/articles/beyond-jmx-supercharging-grafana-dashboards-with-high-fidelity-metrics)

## Dashboard Templates

### 1. Kafka Environment Health

Designed for Platform Teams, this dashboard provides a high-level macro view of overall cluster stability and capacity.

Rather than relying on raw byte counts, it surfaces derived operational health indicators. It tracks total online brokers, overall data on disk, total topics, and total consumer groups. It also visualizes cluster-wide production and consumption rates, and provides a detailed breakdown of topic activity and consumer group health (Stable, Rebalancing, Empty) to give you an instant read on the environment's status.

**🔗 Quick Links:**
* 📥 **JSON Template:** [`kafka-environment.json`](./grafana-templates/kpow/kafka-environment.json)
* 🌐 **Grafana Gallery:** [View and Import Dashboard](https://grafana.com/grafana/dashboards/25103-kafka-environment/) *(ID: 25103)*

### 2. Kafka Topic Diagnostics

Designed for data engineers and platform administrators, this dashboard provides granular visibility into the data layer.

It tracks aggregate metrics like total topics, total replica disk usage, cluster-wide read/write throughput, and non-preferred leaders. Most importantly, it visualizes per-topic production and consumption rates over time, topic size growth, and isolates the exact topics experiencing consumer lag or Under Replicated Partitions (URPs) through detailed diagnostic tables.

**🔗 Quick Links:**
* 📥 **JSON Template:** [`kafka-topic.json`](./grafana-templates/kpow/kafka-topic.json)
* 🌐 **Grafana Gallery:** [View and Import Dashboard](https://grafana.com/grafana/dashboards/25104-kafka-topic/) *(ID: 25104)*

### 3. Kafka Consumer Group Deep Dive

Designed for Application Teams, this dashboard focuses on micro-level Service Level Agreement (SLA) monitoring.

Instead of generic host metrics, it visualizes the exact state of your data consumption. Key metrics include precise total lag (`group_offset_lag`) and real-time consumption rates (`group_offset_delta`). It details total assigned members and hosts, and features a clear status table tracking the exact state of every consumer group to help engineers spot stalling applications before downstream users are impacted.

**🔗 Quick Links:**
* 📥 **JSON Template:** [`kafka-consumer-group.json`](./grafana-templates/kpow/kafka-consumer-group.json)
* 🌐 **Grafana Gallery:** [View and Import Dashboard](https://grafana.com/grafana/dashboards/25105-kafka-consumer-group/) *(ID: 25105)*

### 4. Kafka Connect Operations

Data pipeline reliability depends heavily on integration health. This dashboard targets Kafka Connect deployments, replacing tedious API queries with instant visual feedback.

It tracks aggregate summary statistics alongside individual Connector and Task states. By mapping state labels directly to distinct visual alerts (RUNNING, PAUSED, FAILED, UNASSIGNED, UNREACHABLE), teams can immediately detect stalled integrations and isolate whether the failure exists at the connector or task level.

**🔗 Quick Links:**
* 📥 **JSON Template:** [`kafka-connect.json`](./grafana-templates/kpow/kafka-connect.json)
* 🌐 **Grafana Gallery:** [View and Import Dashboard](https://grafana.com/grafana/dashboards/25106-kafka-connect/) *(ID: 25106)*

## 🚀 Getting Started with Grafana Cloud

These instructions illustrate how to wire up Grafana Cloud's agentless **Metrics Endpoint** integration to scrape Kpow directly, without needing to manage a local Prometheus instance. 

### 📋 Prerequisites: Enable Kpow Telemetry
Before configuring Grafana Cloud, ensure that your Kpow instance is configured to expose its Prometheus metrics and that the endpoints are secured with Basic Authentication (which is strictly required by Grafana Cloud's agentless scraper). 

🔗 **Read the official guide:** [Enabling Kpow's Prometheus Integration](https://docs.factorhouse.io/kpow/integration/prometheus/overview)

### Step 1: Configure Metrics Endpoints (Scrape Jobs)

Grafana Cloud can scrape Kpow directly over the internet. You will need to create a scrape job for each of Kpow's metric endpoints. 

1. Log in to your Grafana Cloud portal.
2. Navigate to **Connections** > **Add new connection**.
3. Search for and select **Metrics Endpoint**.
4. Click **Add new scrape job** and create three separate jobs using the following URLs:
   * `https://<your-kpow-domain>/metrics/v1` (replace with your Kpow domain)
   * `https://<your-kpow-domain>/offsets/v1`
   * `https://<your-kpow-domain>/group-offsets/v1`
   > ❗ **Authentication:** The endpoints should be secured, which is strictly required by the Metrics Endpoint integration. You can select either **Basic** or **Bearer (OAuth)** authentication.
5. Click **Test Connection** and **Save Scrape Job** for each job. Grafana will immediately start polling these endpoints and storing the data in your built-in Prometheus database.

### Step 2: Check Metrics are Flowing

Before importing the dashboards, verify that Grafana Cloud is successfully receiving the data:

1. In Grafana, go to the left-hand menu and click **Explore** (the compass icon).
2. Ensure your default **Prometheus** data source is selected in the top-left dropdown (usually named `grafanacloud-<your-stack>-prom`).
3. In the query bar, type a metric like `topic_count` or `broker_count` and run the query.
4. If you see a graph or data table populate, your connection is working perfectly!

### Step 3: Create Dashboards from Templates

With the data flowing, you can now import the JSON templates provided in this repository.

1. Download the `.json` files from the `grafana-templates/kpow` directory in this repo.
2. In Grafana, navigate to **Dashboards** > **New** > **Import**.
3. Upload the `.json` file (or paste the raw JSON text into the provided box) and click **Load**.
4. At the bottom of the import options screen, you will be prompted to select a **Prometheus** data source. Select your Grafana Cloud Prometheus data source from the dropdown.
5. Click **Import**.

Your dashboard will instantly load and populate with live metrics! Repeat this process for the remaining dashboards.