# 04. Component 2: OtelTraceSink (Span Exporter & Attributes)

## Purpose

`OtelTraceSink` connects ns-3 packet trace events (`PacketTx`, `PacketRx`, `PacketDrop`) to OpenTelemetry `Tracer` instances. It constructs spans tagged with packet attributes, node IDs, event types, and drop reasons.

---

## Class Definition (`model/otlp-trace-sink.h`)

```cpp
#ifndef OTLP_TRACE_SINK_H
#define OTLP_TRACE_SINK_H

#include "ns3/packet.h"
#include "ns3/ptr.h"

#include <opentelemetry/trace/provider.h>
#include <opentelemetry/trace/tracer.h>
#include <string>

namespace ns3 {

/**
 * \ingroup otlp
 * \brief Trace sink mapping ns-3 packet events to OpenTelemetry spans.
 */
class OtelTraceSink
{
public:
  OtelTraceSink();
  ~OtelTraceSink() = default;

  /**
   * \brief Trace a packet transmission event.
   * \param packet The ns-3 packet.
   * \param nodeId ID of transmitting node.
   */
  void TracePacketTx(Ptr<const Packet> packet, uint32_t nodeId);

  /**
   * \brief Trace a packet reception event.
   * \param packet The ns-3 packet.
   * \param nodeId ID of receiving node.
   */
  void TracePacketRx(Ptr<const Packet> packet, uint32_t nodeId);

  /**
   * \brief Trace a packet drop event.
   * \param packet The ns-3 packet.
   * \param nodeId ID of node dropping packet.
   * \param reason Description or reason for packet drop.
   */
  void TracePacketDrop(Ptr<const Packet> packet, uint32_t nodeId, const std::string &reason);
};

} // namespace ns3

#endif // OTLP_TRACE_SINK_H
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
| `ns3.drop_reason` | `string` | Reason for drop (only on drop spans) | `BUFFER_OVERFLOW`, `WIFI_PHY_SNR_LOW` |

---

## Implementation (`model/otlp-trace-sink.cc`)

```cpp
#include "otlp-trace-sink.h"
#include "otlp-clock-provider.h"

#include <opentelemetry/trace/provider.h>
#include <opentelemetry/trace/span_startoptions.h>

namespace ns3 {

OtelTraceSink::OtelTraceSink()
{
}

void
OtelTraceSink::TracePacketTx(Ptr<const Packet> packet, uint32_t nodeId)
{
  auto provider = opentelemetry::trace::Provider::GetTracerProvider();
  auto tracer = provider->GetTracer("ns3-otlp-tracer", "1.0.0");
  if (!tracer || !packet) return;

  opentelemetry::trace::StartSpanOptions options;
  options.start_system_time = OtelClock::GetNow();

  auto span = tracer->StartSpan("packet_tx",
    {
      {"ns3.node_id", static_cast<int64_t>(nodeId)},
      {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
      {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
      {"event.type", "TX_START"}
    },
    options);

  span->End();
}

void
OtelTraceSink::TracePacketRx(Ptr<const Packet> packet, uint32_t nodeId)
{
  auto provider = opentelemetry::trace::Provider::GetTracerProvider();
  auto tracer = provider->GetTracer("ns3-otlp-tracer", "1.0.0");
  if (!tracer || !packet) return;

  opentelemetry::trace::StartSpanOptions options;
  options.start_system_time = OtelClock::GetNow();

  auto span = tracer->StartSpan("packet_rx",
    {
      {"ns3.node_id", static_cast<int64_t>(nodeId)},
      {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
      {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
      {"event.type", "RX_OK"}
    },
    options);

  span->End();
}

void
OtelTraceSink::TracePacketDrop(Ptr<const Packet> packet, uint32_t nodeId, const std::string &reason)
{
  auto provider = opentelemetry::trace::Provider::GetTracerProvider();
  auto tracer = provider->GetTracer("ns3-otlp-tracer", "1.0.0");
  if (!tracer || !packet) return;

  opentelemetry::trace::StartSpanOptions options;
  options.start_system_time = OtelClock::GetNow();

  auto span = tracer->StartSpan("packet_drop",
    {
      {"ns3.node_id", static_cast<int64_t>(nodeId)},
      {"ns3.packet_uid", static_cast<int64_t>(packet->GetUid())},
      {"ns3.packet_size", static_cast<int64_t>(packet->GetSize())},
      {"ns3.drop_reason", reason},
      {"event.type", "DROP"}
    },
    options);

  span->SetStatus(opentelemetry::trace::StatusCode::kError, reason);
  span->End();
}

} // namespace ns3
```
