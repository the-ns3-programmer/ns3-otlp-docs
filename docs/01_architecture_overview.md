# 01. Architecture Overview

## Executive Summary

Network Simulator 3 (ns-3) traditionally exports simulation results into ASCII trace files (`.tr`) or PCAP files (`.pcap`). Analyzing these outputs requires offline post-processing with custom scripts or manual Wireshark inspections.

`ns3-otlp` upgrades ns-3 to the **cloud-native observability standard (OpenTelemetry)**. It enables live, streaming telemetry export during simulation execution, allowing network engineers to visualize real-time packet timelines, distributed trace waterfalls, and performance metrics on web dashboards like **Jaeger UI**, **Grafana**, and **Prometheus**.

---

## High-Level System Architecture

```
+-----------------------------------------------------------------------------------+
|                                  ns-3 Simulator                                   |
|                                                                                   |
|  +--------------------+   TracedCallback   +-----------------------------------+  |
|  | ns-3 Trace Sources | -----------------> |          OtelTraceSink            |  |
|  | (Packet Tx/Rx/Drop)|                    |  (Maps packet events to Spans)   |  |
|  +--------------------+                    +-----------------+-----------------+  |
|                                                              |                    |
|  +--------------------+   FlowMonitor      +-----------------v-----------------+  |
|  | ns-3 Metrics       | -----------------> |          OtelMetricSink           |  |
|  | (Throughput/Delay) |                    | (Maps metrics to Counters/Gauges) |  |
|  +--------------------+                    +-----------------+-----------------+  |
|                                                              |                    |
|                                                    +---------v---------+          |
|                                                    |     OtelClock     |          |
|                                                    | Simulator::Now()  |          |
|                                                    +---------+---------+          |
|                                                              |                    |
+--------------------------------------------------------------|--------------------+
                                                               v
                                             +-----------------------------------+
                                             |     opentelemetry-cpp SDK         |
                                             +-----------------+-----------------+
                                                               | OTLP (HTTP/gRPC)
                                                               v
                                             +-----------------------------------+
                                             |       Jaeger / OTel Collector     |
                                             +-----------------------------------+
```

---

## Core Principles

1. **Virtual Time Synchronization:** Real-world wall-clock timestamps ruin discrete-event simulation telemetry. `OtelClock` maps ns-3 nanoseconds directly to OTLP span timestamps.
2. **Type-Safe Data Mapping:** Translates ns-3 object types (`Ptr<Packet>`, `Ptr<Node>`) into OpenTelemetry attribute key-value maps.
3. **Idiomatic Integration:** Fits into ns-3's existing helper structure (`OtelHelper`).
4. **Zero-Locking Performance:** Emits telemetry asynchronously over gRPC or HTTP to prevent slowing down simulation execution.
