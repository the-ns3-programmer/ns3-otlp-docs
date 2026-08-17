# 04. Component 2: OtelTraceSink (Span Exporter & Attributes)

## Overview & Purpose

`OtelTraceSink` maps ns-3 packet events (`TracePacketTx`, `TracePacketRx`, `TracePacketDrop`) to OpenTelemetry spans. It is implemented as an `ns3::Object` registered with `TypeId` under group `"otlp"`.

---

## Detailed API Specification

Header: `ns3/otlp-trace-sink.h`  
Base Class: `ns3::Object`  
Namespace: `ns3`

### Class Declaration

```cpp
namespace ns3 {

class OtelTraceSink : public Object
{
public:
    static TypeId GetTypeId();

    OtelTraceSink();
    ~OtelTraceSink() override = default;

    /**
     * \brief Fetch active tracer from global Provider and cache m_tracer.
     */
    void InitTracer();

    /**
     * \brief Trace a packet transmission event.
     * \param packet The ns-3 packet being transmitted.
     * \param nodeId ID of the transmitting node.
     */
    void TracePacketTx(Ptr<const Packet> packet, uint32_t nodeId);

    /**
     * \brief Trace a packet reception event.
     * \param packet The ns-3 packet being received.
     * \param nodeId ID of the receiving node.
     */
    void TracePacketRx(Ptr<const Packet> packet, uint32_t nodeId);

    /**
     * \brief Trace a packet drop event.
     * \param packet The ns-3 packet being dropped.
     * \param nodeId ID of the node dropping the packet.
     * \param reason String explanation for the drop.
     */
    void TracePacketDrop(Ptr<const Packet> packet, uint32_t nodeId, const std::string& reason);

private:
    opentelemetry::nostd::shared_ptr<opentelemetry::trace::Tracer> m_tracer;
};

} // namespace ns3
```

---

## Attribute Mapping Dictionary

Each span emitted by `OtelTraceSink` contains standard key-value attributes:

| Attribute Name | Data Type | Description | Example Value |
| :--- | :--- | :--- | :--- |
| `ns3.node_id` | `int64_t` | ID of the active node emitting event | `0`, `1`, `2` |
| `ns3.packet_uid` | `int64_t` | Unique ns-3 Packet UID | `425fd63` |
| `ns3.packet_size` | `int64_t` | Total size of packet in bytes | `1024` |
| `event.type` | `string` | Event category | `TX_START`, `RX_OK`, `DROP` |
| `ns3.drop_reason` | `string` | Reason for drop (only on drop spans) | `PHY_DROP`, `WIFI_PHY_1` |

---

## Implementation Code (`model/otlp-trace-sink.cc`)

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "otlp-trace-sink.h"
#include "otlp-clock-provider.h"

#include "ns3/log.h"
#include <opentelemetry/trace/provider.h>

namespace ns3
{

NS_LOG_COMPONENT_DEFINE("OtelTraceSink");
NS_OBJECT_ENSURE_REGISTERED(OtelTraceSink);

TypeId
OtelTraceSink::GetTypeId()
{
    static TypeId tid = TypeId("ns3::OtelTraceSink")
                            .SetParent<Object>()
                            .SetGroupName("otlp")
                            .AddConstructor<OtelTraceSink>();
    return tid;
}

OtelTraceSink::OtelTraceSink() : m_tracer(nullptr)
{
}

void
OtelTraceSink::InitTracer()
{
    auto provider = opentelemetry::trace::Provider::GetTracerProvider();
    m_tracer = provider->GetTracer("ns3-otlp-tracer", "1.0.0");
    NS_LOG_INFO("OtelTraceSink: tracer initialised");
}

void
OtelTraceSink::TracePacketTx(Ptr<const Packet> packet, uint32_t nodeId)
{
    if (!m_tracer || !packet) return;

    opentelemetry::trace::StartSpanOptions options;
    options.start_system_time = OtelClock::GetNow();

    auto span = m_tracer->StartSpan("packet_tx",
                                    {{"ns3.node_id", static_cast<int64_t>(nodeId)},
                                     {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
                                     {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
                                     {"event.type", "TX_START"}},
                                    options);
    span->End();
}

void
OtelTraceSink::TracePacketRx(Ptr<const Packet> packet, uint32_t nodeId)
{
    if (!m_tracer || !packet) return;

    opentelemetry::trace::StartSpanOptions options;
    options.start_system_time = OtelClock::GetNow();

    auto span = m_tracer->StartSpan("packet_rx",
                                    {{"ns3.node_id", static_cast<int64_t>(nodeId)},
                                     {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
                                     {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
                                     {"event.type", "RX_OK"}},
                                    options);
    span->End();
}

void
OtelTraceSink::TracePacketDrop(Ptr<const Packet> packet, uint32_t nodeId, const std::string& reason)
{
    if (!m_tracer || !packet) return;

    opentelemetry::trace::StartSpanOptions options;
    options.start_system_time = OtelClock::GetNow();

    auto span = m_tracer->StartSpan("packet_drop",
                                    {{"ns3.node_id", static_cast<int64_t>(nodeId)},
                                     {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
                                     {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
                                     {"ns3.drop_reason", reason},
                                     {"event.type", "DROP"}},
                                    options);
    span->SetStatus(opentelemetry::trace::StatusCode::kError, reason);
    span->End();
}

} // namespace ns3
```
