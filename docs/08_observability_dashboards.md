# 08. Observability Dashboards (Docker + Jaeger UI)

## Overview

To visualize telemetry emitted by `ns3-otlp`, a Docker container stack running **Jaeger UI** and **OpenTelemetry Collector** is provided in `contrib/otlp/docker-compose.yml`.

---

## Docker Compose Configuration (`contrib/otlp/docker-compose.yml`)

```yaml
version: '3.8'

services:
  jaeger:
    image: jaegertracing/all-in-one:latest
    container_name: ns3-jaeger
    environment:
      - COLLECTOR_OTLP_ENABLED=true
    ports:
      - "16686:16686" # Jaeger Web UI
      - "4317:4317"   # OTLP gRPC receiver
      - "4318:4318"   # OTLP HTTP receiver
```

---

## Starting the Telemetry Dashboard

```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1/contrib/otlp
docker-compose up -d
```

Check status:
```bash
docker-compose ps
```

---

## Using Jaeger UI

1. Open **`http://localhost:16686`** in any web browser.
2. Under **Service**, select your simulation service:
   - `ns3-basic-p2p-simulation`
   - `ns3-wifi-adhoc-simulation`
3. Click **Find Traces**.

---

## Interpreting Jaeger Visualizations

- **Interactive Scatter Plot:** Shows execution duration of each packet event against simulation time. Teal dots indicate successful packet transmissions and receptions (`packet_tx`, `packet_rx`).
- **Error Highlights (Red Dots):** Red dots highlight packet drops (e.g. `BUFFER_OVERFLOW`, `WIFI_PHY_SNR_LOW`).
- **Span Waterfall View:** Clicking on any trace opens a timeline showing exact microsecond execution durations, Node IDs, Packet UIDs, and attributes.
