# 01. Architecture Overview

## Executive Summary

Network Simulator 3 (ns-3) traditionally exports simulation results into ASCII trace files (`.tr`) or PCAP files (`.pcap`). Analyzing these outputs requires offline post-processing with custom scripts or manual Wireshark inspections.

`ns3-otlp` upgrades ns-3 to the **cloud-native observability standard (OpenTelemetry)**. It enables live, streaming telemetry export during simulation execution, allowing network engineers to visualize real-time packet timelines, distributed trace waterfalls, and performance metrics on web dashboards like **Jaeger UI**.

---

## High-Level System Architecture

```
+-----------------------------------------------------------------------------------+
|                                  ns-3 Simulator                                   |
|                                                                                   |
|  +--------------------+   Trace Trampoline +-----------------------------------+  |
|  | ns-3 Trace Sources | -----------------> |          OtelTraceSink            |  |
|  | (MacTx, MacRx, etc)|                    |    (ns3::Object, cached tracer)   |  |
|  +--------------------+                    +-----------------+-----------------+  |
|                                                              |                    |
|  +--------------------+   Trace Trampoline +-----------------v-----------------+  |
|  | ns-3 NetDevices    | -----------------> |          OtelMetricSink           |  |
|  | (P2P, Wi-Fi)       |                    | (ns3::Object, Counters/Histogram) |  |
|  +--------------------+                    +-----------------+-----------------+  |
|                                                              |                    |
|                                                    +---------v---------+          |
|                                                    |     OtelClock     |          |
|                                                    | wall_base + Sim   |          |
|                                                    +---------+---------+          |
|                                                              |                    |
+--------------------------------------------------------------|--------------------+
                                                               v
                                             +-----------------------------------+
                                             |     opentelemetry-cpp SDK         |
                                             +-----------------+-----------------+
                                                               | OTLP / HTTP
                                                               v
                                             +-----------------------------------+
                                             |       Jaeger / OTel Collector     |
                                             +-----------------------------------+
```

---

## Core Principles

1. **Virtual Time Synchronization (`OtelClock`):** Real-world wall-clock timestamps ruin discrete-event simulation telemetry. `OtelClock` anchors simulation nanoseconds to the real wall clock at `Install()` time (`wall_base + Simulator::Now()`), placing spans at today's date in Jaeger UI.
2. **Type-Safe Data Mapping:** Translates ns-3 object types (`Ptr<Packet>`, `Ptr<Node>`) into OpenTelemetry attribute key-value maps.
3. **Idiomatic ns-3 Integration:** Both `OtelTraceSink` and `OtelMetricSink` inherit from `ns3::Object`, register `TypeId` attributes under group `"otlp"`, and use `Ptr<>` reference-counted pointers managed by `OtelHelper`.
4. **Zero Invasion:** Hooks standard ns-3 trace sources (`MacTx`, `MacRx`, `PhyRxDrop`, `MacTxDrop`) via `TraceConnectWithoutContext` and bound trampoline callbacks without modifying any core ns-3 device source code.
5. **Licensing:** Licensed under **GPL-3.0-or-later**, ensuring full license compatibility with Apache-2.0 third-party dependencies (`opentelemetry-cpp` and `gRPC`).
