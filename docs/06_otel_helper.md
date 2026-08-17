# 06. Component 4: OtelHelper (Orchestration Pipeline)

## Overview & Purpose

`OtelHelper` manages the OpenTelemetry `TracerProvider`, `MeterProvider`, clock anchoring, and trace source installation on ns-3 nodes. It follows a strict 3-phase lifecycle:

$$\text{Setters} \longrightarrow \text{Install()} \longrightarrow \text{EnableNodeTracing()}$$

---

## Detailed API Specification

Header: `helper/otlp-helper.h`  
Namespace: `ns3`

### Class Declaration

```cpp
namespace ns3 {

class OtelHelper
{
public:
    OtelHelper();
    ~OtelHelper() = default;

    /**
     * \brief Set OTLP HTTP collector traces endpoint.
     * \param endpoint URL string (e.g. "http://localhost:4318/v1/traces").
     * \pre Must be called before Install().
     */
    void SetEndpoint(const std::string& endpoint);

    /**
     * \brief Set telemetry service name.
     * \param serviceName Service identifier displayed in backends.
     * \pre Must be called before Install().
     */
    void SetServiceName(const std::string& serviceName);

    /**
     * \brief Configure providers, wall-clock anchor, and initialize sinks.
     */
    void Install();

    /**
     * \brief Connect trace sources on devices for a single node.
     * \param node Ptr to target Node.
     * \pre Must be called after Install().
     */
    void EnableNodeTracing(Ptr<Node> node);

    /**
     * \brief Connect trace sources on devices for a container of nodes.
     * \param nodes NodeContainer of target nodes.
     * \pre Must be called after Install().
     */
    void EnableNodeTracing(NodeContainer nodes);

    Ptr<OtelTraceSink> GetTraceSink() const;
    Ptr<OtelMetricSink> GetMetricSink() const;

private:
    std::string m_endpoint{"http://localhost:4318/v1/traces"};
    std::string m_serviceName{"ns3-simulation"};
    bool m_configured{false};

    Ptr<OtelTraceSink> m_traceSink;
    Ptr<OtelMetricSink> m_metricSink;
};

} // namespace ns3
```

---

## Trace Trampolines & Hook Matrix

`EnableNodeTracing()` dynamically inspects device types attached to each node and connects trampolines via `TraceConnectWithoutContext` and `MakeBoundCallback`:

| Device Type | Trace Source Name | Trampoline Function | Sinks Triggered |
| :--- | :--- | :--- | :--- |
| `PointToPointNetDevice` | `MacTx` | `MacTxCallback` | `TracePacketTx` + `RecordThroughput` |
| `PointToPointNetDevice` | `MacRx` | `MacRxCallback` | `TracePacketRx` + `RecordThroughput` |
| `PointToPointNetDevice` | `PhyRxDrop` | `PhyDropCallback` | `TracePacketDrop("PHY_DROP")` + `RecordPacketDropCount` |
| `PointToPointNetDevice` | `MacTxDrop` | `PhyDropCallback` | `TracePacketDrop("MAC_TX_DROP")` + `RecordPacketDropCount` |
| `WifiNetDevice` | `MacTx` | `MacTxCallback` | `TracePacketTx` + `RecordThroughput` |
| `WifiNetDevice` | `MacRx` | `MacRxCallback` | `TracePacketRx` + `RecordThroughput` |
| `WifiNetDevice` | `PhyRxDrop` | `WifiPhyDropCallback` | `TracePacketDrop("WIFI_PHY_<reason>")` + `RecordPacketDropCount` |
