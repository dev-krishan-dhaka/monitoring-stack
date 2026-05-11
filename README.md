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
|              YOUR CONTAINERS                     |
|        (Backend, Frontend, Database)             |
+----------+-------------+------------+-----------+
           |             |            |
           v             v            v
+----------+  +----------+  +--------+  +---------+
| CADVISOR |  | PROMTAIL |  |BLACKBOX|  | /metrics|
| Metrics  |  |   Logs   |  | Uptime |  | endpoint|
+----+-----+  +----+-----+  +---+----+  +----+----+
     |              |           |            |
     v              v           |            v
+----+-----+  +----+-----+      |       +----+----+
|PROMETHEUS|  |   LOKI   |      |       |PROMETHEUS|
| Storage  |  | Storage  |      |       | Storage  |
+----+-----+  +----+-----+      |       +----+----+
     |              |           |            |
     +--------------+-----------+------------+
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
├── Grafana-dashboard.json
└── README.md
```

---

## Setup

### Step 1 — Clone Repository

```bash
git clone https://github.com/dev-krishan-dhaka/monitoring-stack
cd monitoring-stack
```

### Step 2 — Create Shared Docker Network

```bash
docker network create monitoring
```

This network allows the monitoring stack and your application containers to communicate.

### Step 3 — Start Monitoring Stack

```bash
docker compose up -d
```

Verify running containers:

```bash
docker ps
```

You should see: `grafana`, `prometheus`, `loki`, `promtail`, `cadvisor`, `blackbox-exporter`, `node-exporter`

### Step 4 — Open Grafana

Go to: [http://localhost:3001](http://localhost:3001)

```
Username: admin
Password: admin
```

### Step 5 — Add Data Sources

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

### Step 6 — Import Dashboard

1. Go to **Dashboards → Import**
2. Upload `Grafana-dashboard.json`
3. Select the **Prometheus** data source
4. Click **Import**

---

## Monitor Any Dockerized Application

### Step 1 — Connect App to Monitoring Network

In your application's `docker-compose.yml`, add the `monitoring` network to each service:

```yaml
frontend:
  build: .
  ports:
    - "3000:3000"
  networks:
    - monitoring
```

### Step 2 — Declare External Network

At the bottom of your application's `docker-compose.yml`:

```yaml
networks:
  frontend-network:
  backend-network:
  monitoring:
    external: true
```
### step 3 - Create network 

Add this at the end of compose file: 
```bash
networks:
  monitoring:
    name: monitoring
```

### Step 4 — Start Your Application

If using a build:

```bash
docker compose up --build -d
```

If using a pre-built image:

```bash
docker compose up -d
```

### Step 5 — Configure Blackbox Monitoring

In `blackbox-targets.yml` --

```yaml
  - targets:
      - 'http://frontend:3000'
      - 'http://backend:5000/health'
```

> **Important:** Use Docker service names, not `localhost`.
> ✅ `http://frontend:3000`
> ❌ `http://localhost:3000`

### Step 6 — Restart Prometheus

```bash
docker compose restart prometheus
```

# Example - 




## Step 1 — Clone and Configure the Application

### Clone the repository

```bash
git clone https://github.com/docker/awesome-compose
cd awesome-compose/react-express-mongodb
```

### Update `compose.yaml`

Edit the `compose.yaml` file to add all services to the `monitoring` network.

**Frontend service:**

```yaml
networks:
  - react-express
  - monitoring
```

**Backend service:**

```yaml
networks:
  - express-mongo
  - react-express
  - monitoring
```

**Database service:**

```yaml
networks:
  - express-mongo
  - monitoring
```

**At the bottom of `compose.yaml`, update the networks section:**

```yaml
networks:
  monitoring:
    name: monitoring
```

### Start the application

```bash
docker compose up --build -d
```

---

## Step 3 — Clone and Configure the Monitoring Stack

### Clone the monitoring stack

```bash
git clone https://github.com/dev-krishan-dhaka/monitoring-stack
cd monitoring-stack
```

### Update `blackbox-target.yml`

```yaml
- targets:
    - 'http://frontend:3000'
    - 'http://backend:5000/health'
```

### Start the monitoring stack

```bash
docker compose up -d
```

---

## Step 4 — Set Up Grafana Dashboard

1. Open Grafana in your browser (default: `http://localhost:3001`)
2. Navigate to **Dashboards → Import**
3. Upload or paste the contents of `Grafana-dashboard-sample.json`
4. Click **Import** to load the dashboard

---

## Verify Monitoring

### Prometheus Targets

Open [http://localhost:9090/targets](http://localhost:9090/targets) — all targets should show **UP**.

### Useful Queries

| What | Query |
|------|-------|
| Uptime check | `probe_success` |
| Response time | `probe_duration_seconds` |
| Container CPU | `rate(container_cpu_usage_seconds_total[1m]) * 100` |
| Container Memory (MB) | `container_memory_working_set_bytes / 1024 / 1024` |

### Loki Log Queries (Grafana → Explore → Loki)

| What | Query |
|------|-------|
| All logs | `{job="docker"}` |
| Frontend logs | `{container=~".*frontend.*"}` |
| Backend logs | `{container=~".*backend.*"}` |
| MongoDB logs | `{container=~".*mongo.*"}` |
| Postgres logs | `{container=~".*postgres.*"}` |

---

## Grafana Dashboard Structure

| Row | Panels |
|-----|--------|
| Frontend | Status · CPU · Memory |
| Backend | Status · CPU · Memory |
| Database | Status · CPU · Memory |
| Logs | All · Frontend · Backend · Database |

---

## Prometheus Queries Reference

**Frontend Status**
```promql
probe_success{instance="http://frontend:3000"}
```

**Backend Status**
```promql
probe_success{instance="http://backend:5000/health"}
```

**Frontend CPU**
```promql
rate(container_cpu_usage_seconds_total{name=~".*frontend.*"}[1m]) * 100
```

**Backend CPU**
```promql
rate(container_cpu_usage_seconds_total{name=~".*backend.*"}[1m]) * 100
```

**Frontend Memory**
```promql
container_memory_working_set_bytes{name=~".*frontend.*"} / 1024 / 1024
```

**Backend Memory**
```promql
container_memory_working_set_bytes{name=~".*backend.*"} / 1024 / 1024
```

---

## Troubleshooting

**Port already allocated**

```bash
docker compose down
# or
docker stop <container_name>
```

**No logs in Loki**

```bash
docker compose restart promtail
docker logs monitoring-stack-promtail-1
```

**No metrics in Grafana**

Check [http://localhost:9090/targets](http://localhost:9090/targets) — all targets must be **UP**.

**Blackbox status shows 0**

```bash
curl http://frontend:3000
# or
curl http://backend:5000/health
```

---

## Stack Management

```bash
# Stop stack
docker compose down

# Restart stack
docker compose restart
```

---

## License

MIT
