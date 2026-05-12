# Docker Monitoring Stack

A reusable observability stack for monitoring any Dockerized application.

**Stack:** Prometheus · Grafana · Loki · Promtail · cAdvisor · Blackbox Exporter · Node Exporter

---

## Features

- Container CPU & memory monitoring
- Container uptime monitoring
- Application health checks
- Docker logs collection
- Grafana dashboards
- Real-time observability
- Works with any Docker Compose app

---

## Architecture

```
+--------------------------------------------------+
|                 CONTAINERS                       |
|        (Backend, Frontend, Database)             |
+----------+----------+----------+----------------+
           |          |          |          |
           v          v          v          v
+----------+ +--------+ +--------+ +---------+
| CADVISOR | |PROMTAIL| |BLACKBOX| | /metrics|
| Metrics  | |  Logs  | | Uptime | | endpoint|
+----+-----+ +---+----+ +---+----+ +----+----+
     |            |          |           |
     v            v          |           v
+----------+ +--------+      |      +----------+
|PROMETHEUS| |  LOKI  |      |      |PROMETHEUS|
| Storage  | |Storage |      |      | Storage  |
+----+-----+ +---+----+      |      +----+-----+
     |            |          |           |
     +------------+----------+-----------+
                        |
                        v
               +------------------+
               |     GRAFANA      |
               | Dashboards & UI  |
               +------------------+
```




## Requirements

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)

Verify installation:

```bash
docker --version
docker compose version
```

---

## Project Structure

```
monitoring-stack/
├── docker-compose.yml
├── prometheus.yml
├── promtail-config.yaml
├── Grafana-dashboard-Sample.json
└── README.md
```

---

## Setup

### Step 1 — Clone Repository

```bash
git clone https://github.com/dev-krishan-dhaka/monitoring-stack
cd monitoring-stack
```


### Step 2 — Start Monitoring Stack

```bash
docker compose up -d
```

Verify running containers:

```bash
docker ps
```

You should see: `grafana`, `prometheus`, `loki`, `promtail`, `cadvisor`, `blackbox-exporter`, `node-exporter`

### Step 3 — Open Grafana

Go to: [http://localhost:3001](http://localhost:3001)

```
Username: admin
Password: admin
```

### Step 4 — Add Data Sources

**Prometheus**

1. Go to **Connections → Data Sources → Add data source**
2. Select **Prometheus**
3. URL: `http://prometheus:9090`
4. Click **Save & Test**

**Loki**

1. Add another data source
2. Select **Loki**
3. URL: `http://loki:3100`
4. Click **Save & Test**

### Step 5 — Import Dashboard

1. Go to **Dashboards → Import**
2. Upload `Grafana-dashboard-Sample.json`
3. Select the **Prometheus** data source
4. Click **Import**

---

## Monitor Any Dockerized Application


### Step 1 — Start the Application

```bash
docker compose up --build -d
```

---

### Step 2 — Ensure Ports Are Exposed

Your application's `docker-compose.yml` must expose ports to the **host machine**:

```yaml
ports:
  - "3000:3000"
```

---

### Step 3 — Configure Blackbox Targets

Edit `blackbox-targets.yml` in the monitoring stack directory:

```yaml
- targets:
    - http://host.docker.internal:3000
```

> ✅ **Always use** `host.docker.internal` with the exposed host port.
>
> ❌ **Do NOT use** internal Docker service names like `http://frontend:3000` — Blackbox Exporter cannot reach them across networks.

---

### Step 4 — Restart Prometheus

```bash
docker compose restart prometheus
```

---

### Step 5 — Verify Targets

Open Prometheus in your browser:

```
http://localhost:9090/targets
```

All configured targets should show status: **`UP`**

---

## Example — React + Express + MongoDB App

A complete walkthrough using the [Docker Awesome Compose](https://github.com/docker/awesome-compose/tree/master/react-express-mongodb) example.

### Step 1 — Clone the Example App

```bash
git clone https://github.com/docker/awesome-compose
cd awesome-compose/react-express-mongodb
```

### Step 2 — Start the Application

```bash
docker compose up --build -d
```

Frontend will be available at: `http://localhost:3000`

### Step 3 — Configure Blackbox Monitoring

Navigate to your monitoring stack directory and edit `blackbox-targets.yml`:

```yaml
- targets:
    - http://host.docker.internal:3000
```

### Step 4 — Restart Prometheus

```bash
docker compose restart prometheus
```

### Step 5 — Open Grafana

```
http://localhost:3001
```

| Field    | Value   |
|----------|---------|
| Username | `admin` |
| Password | `admin` |

### Step 6 — Import Dashboard

1. Go to **Dashboards → Import**
2. Upload `Grafana-dashboard.json`

---

## Useful Prometheus Queries

### Frontend HTTP Status

```promql
probe_success{instance="http://host.docker.internal:3000"}
```

### Frontend CPU Usage (%)

```promql
rate(container_cpu_usage_seconds_total{
  container_label_com_docker_compose_service="frontend"
}[1m]) * 100
```

### Frontend Memory Usage (MB)

```promql
container_memory_working_set_bytes{
  container_label_com_docker_compose_service="frontend"
} / 1024 / 1024
```

### Backend CPU Usage (%)

```promql
rate(container_cpu_usage_seconds_total{
  container_label_com_docker_compose_service="backend"
}[1m]) * 100
```

### Backend Memory Usage (MB)

```promql
container_memory_working_set_bytes{
  container_label_com_docker_compose_service="backend"
} / 1024 / 1024
```

---

## Useful Loki Log Queries

### All Container Logs

```logql
{job="docker"}
```

### Frontend Logs

```logql
{container=~".*frontend.*"}
```

### Backend Logs

```logql
{container=~".*backend.*"}
```

### MongoDB Logs

```logql
{container=~".*mongo.*"}
```

### Error Logs

```logql
{job="docker"} |= "error"
```

### Warning Logs

```logql
{job="docker"} |= "warn"
```

---

## Recommended Dashboard Structure

| Row       | Panels                          |
|-----------|---------------------------------|
| Frontend  | Status · CPU · Memory           |
| Backend   | CPU · Memory · Logs             |
| Database  | CPU · Memory                    |
| Logs      | All Logs · Errors · Warnings    |

---

## 📁 File Reference

| File                    | Purpose                                      |
|-------------------------|----------------------------------------------|
| `docker-compose.yml`    | Monitoring stack services definition         |
| `blackbox-targets.yml`  | HTTP endpoints to probe                      |
| `prometheus.yml`        | Prometheus scrape configuration              |
| `Grafana-dashboard.json`| Pre-built Grafana dashboard to import        |


## License

MIT
