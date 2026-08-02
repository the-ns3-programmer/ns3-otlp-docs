# ns3-otlp Documentation Manual

Welcome to the official technical documentation for **`ns3-otlp`**, an open-source C++ extension module for Network Simulator 3 (ns-3) that exports real-time simulation traces and performance metrics over OTLP (OpenTelemetry Protocol, gRPC/HTTP) directly to cloud-native observability backends including Jaeger UI, Grafana, and Prometheus.

---

## Table of Contents

1. [Architecture Overview](01_architecture_overview.md)
2. [Prerequisites & Environment Setup](02_prerequisites_and_setup.md)
3. [Component 1: OtelClock (Virtual Time Synchronization)](03_otel_clock.md)
4. [Component 2: OtelTraceSink (Span Exporter & Attributes)](04_otel_trace_sink.md)
5. [Component 3: OtelMetricSink (Counters & Gauges)](05_otel_metric_sink.md)
6. [Component 4: OtelHelper (Initialization & Exporter Pipeline)](06_otel_helper.md)
7. [Tutorials & Simulation Examples](07_tutorials_and_examples.md)
8. [Observability Dashboards (Docker + Jaeger UI)](08_observability_dashboards.md)
9. [Automated Unit Testing Framework](09_unit_testing.md)
10. [Troubleshooting & FAQ](10_troubleshooting_faq.md)

---

## Project Metadata

- **Version:** 1.0.0
- **Author & Maintainer:** Arun Santhosh R A (arunsanthosh.rashok@gmail.com)
- **License:** GNU General Public License v2.0 (GPLv2)
- **Target Simulator:** ns-3.36+ (tested on ns-3.46.1)
- **C++ Standard:** C++17 / C++20 / C++23
