# ns3-otlp Documentation

Welcome to the official technical documentation for **ns3-otlp**, a native OpenTelemetry extension module for Network Simulator 3 (ns-3).

`ns3-otlp` bridges discrete-event network simulation with cloud-native observability by exporting real simulation traces and performance metrics over OTLP/HTTP to backends such as **Jaeger UI**.

---

## Navigation Directory

### Architecture & Setup
- [**01. Architecture Overview**](01_architecture_overview.md) — High-level system design, virtual clock anchoring (`OtelClock`), ns-3 `Object`/`TypeId` integration, trace trampolines, and OTLP exporter pipeline.
- [**02. Prerequisites & Environment Setup**](02_prerequisites_and_setup.md) — System requirements, installing dependencies, building `opentelemetry-cpp` from source, configuring ns-3, and running Jaeger.

### API Reference & Core Components
- [**03. OtelClock (Virtual Time)**](03_otel_clock.md) — Wall-clock anchored time conversion API and nanosecond timestamp synchronization.
- [**04. OtelTraceSink (Span Exporter)**](04_otel_trace_sink.md) — OpenTelemetry trace span exporter API, attribute dictionary, and span lifecycle.
- [**05. OtelMetricSink (Metrics)**](05_otel_metric_sink.md) — Metrics exporter API covering throughput counters, drop counters, and queue occupancy histograms.
- [**06. OtelHelper (Pipeline)**](06_otel_helper.md) — Helper orchestrator API, lifecycle management, and trace source trampoline hooks for P2P and Wi-Fi devices.

### Guides, Testing & Troubleshooting
- [**07. Tutorials & Simulation Examples**](07_tutorials_and_examples.md) — Complete C++ simulation code for Point-to-Point UDP Echo and 802.11b Wi-Fi Ad-hoc examples.
- [**08. Observability Dashboards**](08_observability_dashboards.md) — Running Jaeger UI, searching trace waterfalls, and verifying span attributes.
- [**09. Automated Unit Testing**](09_unit_testing.md) — Offline unit testing with `InMemorySpanExporter`.
- [**10. Troubleshooting & FAQ**](10_troubleshooting_faq.md) — Resolving build issues, pkg-config paths, Docker commands, and common questions.

---

## Quick Start Summary

```bash
# 1. Start Jaeger UI & OTLP HTTP receiver (Port 4318)
docker run -d --name jaeger \
  -p 4318:4318 \
  -p 16686:16686 \
  jaegertracing/all-in-one:1.57

# 2. Configure & build ns-3 with OTLP module
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 configure --enable-examples --enable-tests
./ns3 build otlp

# 3. Run unit tests (offline, no network required)
./ns3 run "test-runner --suite=otlp --verbose"

# 4. Run interactive P2P simulation example
./ns3 run otlp-basic-example

# 5. Open Jaeger UI in your browser: http://localhost:16686
```

---

## Module Metadata

- **Version:** 1.0.0
- **License:** GNU General Public License v3.0 or later (`GPL-3.0-or-later`)
- **SPDX Identifier:** `GPL-3.0-or-later`
- **Target Simulator:** ns-3 (tested on ns-3.46.1)
- **OTel SDK Compatibility:** `opentelemetry-cpp` ≥ 1.13.0
- **Author:** Arun Santhosh R A (<arunsanthosh.rashok@gmail.com>)
