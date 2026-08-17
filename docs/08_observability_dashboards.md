# 08. Observability Dashboards (Jaeger UI)

This guide describes viewing and analyzing exported simulation telemetry in Jaeger UI.

---

## 1. Starting Jaeger UI

Run Jaeger using Docker:

```bash
docker run -d --name jaeger \
  -p 4318:4318 \
  -p 16686:16686 \
  jaegertracing/all-in-one:1.57
```

- **OTLP HTTP Traces Endpoint:** `http://localhost:4318/v1/traces`
- **Jaeger Web UI:** `http://localhost:16686`

---

## 2. Searching and Inspecting Spans

1. Open **http://localhost:16686** in your browser.
2. In the left navigation panel under **Service**, select your configured service name:
   - `ns3-p2p-telemetry-demo` (from `otlp-basic-example`)
   - `ns3-wifi-adhoc-simulation` (from `otlp-wifi-example`)
3. Click **Find Traces**.
4. Spans will display grouped by operation name (`packet_tx`, `packet_rx`, `packet_drop`).
5. Click on any trace to inspect individual packet metadata attributes:
   - `ns3.node_id`
   - `ns3.packet_uid`
   - `ns3.packet_size`
   - `event.type`
   - `ns3.drop_reason` (if applicable)

---

## 3. Timestamp Anchoring Verification

Spans appear under the current wall-clock date/time of execution because `OtelClock` adds `Simulator::Now()` as a virtual offset to the wall-clock baseline captured during `OtelHelper::Install()`.
