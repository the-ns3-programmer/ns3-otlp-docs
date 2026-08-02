# 05. Component 3: OtelMetricSink (Counters & Gauges)

## Purpose

`OtelMetricSink` exports numerical simulation metrics (such as bytes transferred, packet drop counts, and queue occupancy) into OpenTelemetry **Counters** and **Gauges**, enabling quantitative real-time visualization in Prometheus and Grafana.

---

## Class Definition (`model/otlp-metric-sink.h`)

```cpp
#ifndef OTLP_METRIC_SINK_H
#define OTLP_METRIC_SINK_H

#include "ns3/ptr.h"

#include <opentelemetry/metrics/meter.h>
#include <opentelemetry/metrics/provider.h>
#include <opentelemetry/metrics/sync_instruments.h>
#include <string>

namespace ns3 {

/**
 * \ingroup otlp
 * \brief Metric sink updating OpenTelemetry Counters with ns-3 performance data.
 */
class OtelMetricSink
{
public:
  OtelMetricSink();
  ~OtelMetricSink() = default;

  /**
   * \brief Record throughput metric (bytes processed).
   * \param nodeId ID of node.
   * \param bytes Bytes transferred.
   */
  void RecordThroughput(uint32_t nodeId, double bytes);

  /**
   * \brief Record packet drop count metric.
   * \param nodeId ID of node.
   * \param count Drop increment count.
   */
  void RecordPacketDropCount(uint32_t nodeId, uint64_t count = 1);

  /**
   * \brief Record current queue size metric.
   * \param nodeId ID of node.
   * \param packetsInQueue Current queue length.
   */
  void RecordQueueSize(uint32_t nodeId, uint64_t packetsInQueue);

private:
  opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Meter> m_meter;
  opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Counter<uint64_t>> m_dropCounter;
  opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Counter<double>> m_throughputCounter;
  opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Counter<uint64_t>> m_queueCounter;
};

} // namespace ns3

#endif // OTLP_METRIC_SINK_H
```

---

## Implementation (`model/otlp-metric-sink.cc`)

```cpp
#include "otlp-metric-sink.h"

namespace ns3 {

OtelMetricSink::OtelMetricSink()
{
  auto provider = opentelemetry::metrics::Provider::GetMeterProvider();
  m_meter = provider->GetMeter("ns3-otlp-meter", "1.0.0");

  if (m_meter)
  {
    m_dropCounter = m_meter->CreateUInt64Counter("ns3.packet_drops", "Number of dropped packets", "packets");
    m_throughputCounter = m_meter->CreateDoubleCounter("ns3.bytes_transferred", "Total bytes transferred", "bytes");
    m_queueCounter = m_meter->CreateUInt64Counter("ns3.queue_events", "Queue length events", "packets");
  }
}

void
OtelMetricSink::RecordThroughput(uint32_t nodeId, double bytes)
{
  if (m_throughputCounter)
  {
    m_throughputCounter->Add(bytes, {{"ns3.node_id", static_cast<int64_t>(nodeId)}});
  }
}

void
OtelMetricSink::RecordPacketDropCount(uint32_t nodeId, uint64_t count)
{
  if (m_dropCounter)
  {
    m_dropCounter->Add(count, {{"ns3.node_id", static_cast<int64_t>(nodeId)}});
  }
}

void
OtelMetricSink::RecordQueueSize(uint32_t nodeId, uint64_t packetsInQueue)
{
  if (m_queueCounter)
  {
    m_queueCounter->Add(packetsInQueue, {{"ns3.node_id", static_cast<int64_t>(nodeId)}});
  }
}

} // namespace ns3
```
