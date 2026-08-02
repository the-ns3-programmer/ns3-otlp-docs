# 07. Tutorials & Simulation Examples

This section presents two complete simulation examples demonstrating how to use `ns3-otlp` in Point-to-Point and Wireless Ad-Hoc networks.

---

## Example 1: Point-to-Point UDP Simulation (`otlp-basic-example.cc`)

### Code Overview (`contrib/otlp/examples/otlp-basic-example.cc`)

```cpp
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/point-to-point-module.h"
#include "ns3/applications-module.h"
#include "ns3/otlp-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("OtelBasicExample");

int main(int argc, char *argv[])
{
  CommandLine cmd(__FILE__);
  cmd.Parse(argc, argv);

  Time::SetResolution(Time::NS);

  LogComponentEnable("OtelBasicExample", LOG_LEVEL_INFO);
  NS_LOG_INFO("Initializing ns3-otlp simulation...");

  // 1. Initialize OpenTelemetry Exporter Helper
  OtelHelper otel;
  otel.SetServiceName("ns3-p2p-telemetry-demo");

  // 2. Create nodes
  NodeContainer nodes;
  nodes.Create(2);

  // 3. Setup Point-to-Point Link
  PointToPointHelper pointToPoint;
  pointToPoint.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
  pointToPoint.SetChannelAttribute("Delay", StringValue("2ms"));

  NetDeviceContainer devices = pointToPoint.Install(nodes);

  // 4. Install Internet Stack
  InternetStackHelper stack;
  stack.Install(nodes);

  Ipv4AddressHelper address;
  address.SetBase("10.1.1.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces = address.Assign(devices);

  // 5. Install Applications
  uint16_t port = 9;
  UdpEchoServerHelper echoServer(port);
  ApplicationContainer serverApps = echoServer.Install(nodes.Get(1));
  serverApps.Start(Seconds(1.0));
  serverApps.Stop(Seconds(10.0));

  UdpEchoClientHelper echoClient(interfaces.GetAddress(1), port);
  echoClient.SetAttribute("MaxPackets", UintegerValue(10));
  echoClient.SetAttribute("Interval", TimeValue(Seconds(1.0)));
  echoClient.SetAttribute("PacketSize", UintegerValue(1024));

  ApplicationContainer clientApps = echoClient.Install(nodes.Get(0));
  clientApps.Start(Seconds(2.0));
  clientApps.Stop(Seconds(10.0));

  otel.EnableNodeTracing(nodes);

  // Emit a series of packet spans simulating full network activity
  auto traceSink = otel.GetTraceSink();
  auto metricSink = otel.GetMetricSink();

  for (uint32_t pktId = 1; pktId <= 10; ++pktId)
  {
    Ptr<Packet> pkt = Create<Packet>(1024);
    traceSink->TracePacketTx(pkt, 0);       // Node 0 transmits
    metricSink->RecordThroughput(0, 1024.0);

    if (pktId == 5)
    {
      // Simulate a packet drop at Node 0 for demonstration
      traceSink->TracePacketDrop(pkt, 0, "BUFFER_OVERFLOW");
      metricSink->RecordPacketDropCount(0, 1);
    }
    else
    {
      traceSink->TracePacketRx(pkt, 1);     // Node 1 receives
      metricSink->RecordThroughput(1, 1024.0);
    }
  }

  NS_LOG_INFO("Running simulation...");
  Simulator::Stop(Seconds(10.0));
  Simulator::Run();

  Simulator::Destroy();
  NS_LOG_INFO("Simulation finished successfully!");

  return 0;
}
```

### How to Run:
```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 run otlp-basic-example
```

---

## Example 2: 802.11b Wireless Ad-Hoc Simulation (`otlp-wifi-example.cc`)

### Code Overview (`contrib/otlp/examples/otlp-wifi-example.cc`)

```cpp
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/internet-module.h"
#include "ns3/wifi-module.h"
#include "ns3/mobility-module.h"
#include "ns3/applications-module.h"
#include "ns3/otlp-module.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE("OtelWifiExample");

int main(int argc, char *argv[])
{
  CommandLine cmd(__FILE__);
  cmd.Parse(argc, argv);

  Time::SetResolution(Time::NS);

  LogComponentEnable("OtelWifiExample", LOG_LEVEL_INFO);
  NS_LOG_INFO("Initializing ns3-otlp Wi-Fi simulation...");

  // 1. Initialize OpenTelemetry Exporter Helper with custom Service Name
  OtelHelper otel("ns3-wifi-adhoc-simulation");

  // 2. Create 3 Wireless Nodes
  NodeContainer wifiNodes;
  wifiNodes.Create(3);

  // 3. Configure Wi-Fi PHY and Channel
  YansWifiChannelHelper channel = YansWifiChannelHelper::Default();
  YansWifiPhyHelper phy;
  phy.SetChannel(channel.Create());

  // 4. Configure Wi-Fi MAC (Ad-Hoc Mode)
  WifiMacHelper mac;
  WifiHelper wifi;
  wifi.SetStandard(WIFI_STANDARD_80211b);
  wifi.SetRemoteStationManager("ns3::ConstantRateWifiManager");

  mac.SetType("ns3::AdhocWifiMac");
  NetDeviceContainer wifiDevices = wifi.Install(phy, mac, wifiNodes);

  // 5. Configure Mobility (Nodes in a line: Node 0 at 0m, Node 1 at 50m, Node 2 at 100m)
  MobilityHelper mobility;
  Ptr<ListPositionAllocator> positionAlloc = CreateObject<ListPositionAllocator>();
  positionAlloc->Add(Vector(0.0, 0.0, 0.0));   // Node 0
  positionAlloc->Add(Vector(50.0, 0.0, 0.0));  // Node 1
  positionAlloc->Add(Vector(100.0, 0.0, 0.0)); // Node 2
  mobility.SetPositionAllocator(positionAlloc);
  mobility.SetMobilityModel("ns3::ConstantPositionMobilityModel");
  mobility.Install(wifiNodes);

  // 6. Install Internet Stack
  InternetStackHelper stack;
  stack.Install(wifiNodes);

  Ipv4AddressHelper address;
  address.SetBase("192.168.1.0", "255.255.255.0");
  Ipv4InterfaceContainer interfaces = address.Assign(wifiDevices);

  // 7. Setup UDP Echo Server on Node 2
  uint16_t port = 9;
  UdpEchoServerHelper echoServer(port);
  ApplicationContainer serverApps = echoServer.Install(wifiNodes.Get(2));
  serverApps.Start(Seconds(1.0));
  serverApps.Stop(Seconds(10.0));

  // 8. Setup UDP Echo Client on Node 0
  UdpEchoClientHelper echoClient(interfaces.GetAddress(2), port);
  echoClient.SetAttribute("MaxPackets", UintegerValue(15));
  echoClient.SetAttribute("Interval", TimeValue(Seconds(0.5)));
  echoClient.SetAttribute("PacketSize", UintegerValue(1024));

  ApplicationContainer clientApps = echoClient.Install(wifiNodes.Get(0));
  clientApps.Start(Seconds(2.0));
  clientApps.Stop(Seconds(10.0));

  otel.EnableNodeTracing(wifiNodes);

  // Emit Wi-Fi Telemetry Spans & Metrics
  auto traceSink = otel.GetTraceSink();
  auto metricSink = otel.GetMetricSink();

  for (uint32_t pktId = 1; pktId <= 15; ++pktId)
  {
    Ptr<Packet> pkt = Create<Packet>(1024);

    // Node 0 Wireless Transmission
    traceSink->TracePacketTx(pkt, 0);
    metricSink->RecordThroughput(0, 1024.0);

    // Node 1 Hop Reception
    traceSink->TracePacketRx(pkt, 1);
    metricSink->RecordThroughput(1, 1024.0);

    if (pktId == 7 || pktId == 12)
    {
      // Simulate Wi-Fi Interference / SNR Drop at Node 2
      traceSink->TracePacketDrop(pkt, 2, "WIFI_PHY_SNR_LOW");
      metricSink->RecordPacketDropCount(2, 1);
    }
    else
    {
      // Node 2 Final Reception
      traceSink->TracePacketRx(pkt, 2);
      metricSink->RecordThroughput(2, 1024.0);
    }
  }

  NS_LOG_INFO("Running Wi-Fi simulation...");
  Simulator::Stop(Seconds(10.0));
  Simulator::Run();

  Simulator::Destroy();
  NS_LOG_INFO("Wi-Fi simulation finished successfully!");

  return 0;
}
```

### How to Run:
```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 run otlp-wifi-example
```
