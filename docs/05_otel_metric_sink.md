# 05. Component 3: OtelMetricSink (Counters & Queue Histogram)

## Overview & Purpose

`OtelMetricSink` exports ns-3 metric data over OTLP. It defines monotonic counters for cumulative bytes (`ns3.bytes_transferred`) and packet drop counts (`ns3.packet_drops`), and a `Histogram` for non-monotonic queue occupancy samples (`ns3.queue_size`).

---

## Detailed API Specification

Header: `ns3/otlp-metric-sink.h`  
Base Class: `ns3::Object`  
Namespace: `ns3`

### Class Declaration

```cpp
namespace ns3 {

class OtelMetricSink : public Object
{
public:
    static TypeId GetTypeId();

    OtelMetricSink();
    ~OtelMetricSink() override = default;

    /**
     * \brief Fetch active meter from global Provider and create instruments.
     */
    void InitMeter();

    /**
     * \brief Record throughput in bytes.
     * \param nodeId Node ID.
     * \param bytes Number of bytes transferred.
     */
    void RecordThroughput(uint32_t nodeId, double bytes);

    /**
     * \brief Record packet drop count.
     * \param nodeId Node ID.
     * \param count Number of dropped packets.
     */
    void RecordPacketDropCount(uint32_t nodeId, uint64_t count = 1);

    /**
     * \brief Record queue occupancy sample (Histogram).
     * \param nodeId Node ID.
     * \param packetsInQueue Current number of packets in queue.
     */
    void RecordQueueSize(uint32_t nodeId, uint64_t packetsInQueue);

private:
    opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Meter> m_meter;
    opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Counter<uint64_t>> m_dropCounter;
    opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Counter<double>> m_throughputCounter;
    opentelemetry::nostd::shared_ptr<opentelemetry::metrics::Histogram<uint64_t>> m_queueHistogram;
};

} // namespace ns3
```

---

## Metric Instruments Summary

| Metric Name | Instrument Type | Units | Description |
| :--- | :--- | :--- | :--- |
| `ns3.bytes_transferred` | `Counter<double>` | `bytes` | Monotonic sum of transferred bytes per node |
| `ns3.packet_drops` | `Counter<uint64_t>` | `packets` | Monotonic count of dropped packets per node |
| `ns3.queue_size` | `Histogram<uint64_t>` | `packets` | Point-in-time queue occupancy samples per node |

---

## Implementation Code (`model/otlp-metric-sink.cc`)

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "otlp-metric-sink.h"
#include "ns3/log.h"
#include <opentelemetry/context/context.h>

namespace ns3
{

NS_LOG_COMPONENT_DEFINE("OtelMetricSink");
NS_OBJECT_ENSURE_REGISTERED(OtelMetricSink);

TypeId
OtelMetricSink::GetTypeId()
{
    static TypeId tid = TypeId("ns3::OtelMetricSink")
                            .SetParent<Object>()
                            .SetGroupName("otlp")
                            .AddConstructor<OtelMetricSink>();
    return tid;
}

OtelMetricSink::OtelMetricSink()
    : m_meter(nullptr),
      m_dropCounter(nullptr),
      m_throughputCounter(nullptr),
      m_queueHistogram(nullptr)
{
}

void
OtelMetricSink::InitMeter()
{
    auto provider = opentelemetry::metrics::Provider::GetMeterProvider();
    m_meter = provider->GetMeter("ns3-otlp-meter", "1.0.0");

    if (m_meter)
    {
        m_dropCounter = m_meter->CreateUInt64Counter("ns3.packet_drops",
                                                     "Number of dropped packets",
                                                     "packets");
        m_throughputCounter = m_meter->CreateDoubleCounter("ns3.bytes_transferred",
                                                           "Total bytes transferred",
                                                           "bytes");
        m_queueHistogram =
            m_meter->CreateUInt64Histogram("ns3.queue_size", "Queue occupancy samples", "packets");
    }

    NS_LOG_INFO("OtelMetricSink: meter initialised");
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
    if (m_queueHistogram)
    {
        m_queueHistogram->Record(packetsInQueue,
                                 {{"ns3.node_id", static_cast<int64_t>(nodeId)}},
                                 opentelemetry::context::Context{});
    }
}

} // namespace ns3
```
