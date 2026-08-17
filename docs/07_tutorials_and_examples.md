# 07. Tutorials & Simulation Examples

This guide walks through using `ns3-otlp` in C++ simulation scripts, featuring the Point-to-Point UDP echo example and the 802.11b Wi-Fi ad-hoc example.

---

## Example 1: Point-to-Point UDP Echo (`examples/otlp-basic-example.cc`)

### Code Overview

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/network-module.h"
#include "ns3/otlp-module.h"
#include "ns3/point-to-point-module.h"

using namespace ns3;

int main(int argc, char* argv[])
{
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

    Time::SetResolution(Time::NS);

    // 1. Set up OtelHelper
    OtelHelper otel;
    otel.SetEndpoint("http://localhost:4318/v1/traces");
    otel.SetServiceName("ns3-p2p-telemetry-demo");
    otel.Install();

    // 2. Build topology
    NodeContainer nodes;
    nodes.Create(2);

    PointToPointHelper p2p;
    p2p.SetDeviceAttribute("DataRate", StringValue("5Mbps"));
    p2p.SetChannelAttribute("Delay", StringValue("2ms"));
    NetDeviceContainer devices = p2p.Install(nodes);

    InternetStackHelper stack;
    stack.Install(nodes);

    Ipv4AddressHelper address;
    address.SetBase("10.1.1.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces = address.Assign(devices);

    // 3. Applications
    uint16_t port = 9;
    UdpEchoServerHelper echoServer(port);
    ApplicationContainer serverApps = echoServer.Install(nodes.Get(1));
    serverApps.Start(Seconds(1.0));
    serverApps.Stop(Seconds(11.0));

    UdpEchoClientHelper echoClient(interfaces.GetAddress(1), port);
    echoClient.SetAttribute("MaxPackets", UintegerValue(10));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(1.0)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(nodes.Get(0));
    clientApps.Start(Seconds(2.0));
    clientApps.Stop(Seconds(11.0));

    // 4. Hook real trace sources
    otel.EnableNodeTracing(nodes);

    Simulator::Stop(Seconds(12.0));
    Simulator::Run();
    Simulator::Destroy();

    return 0;
}
```

### Running the Example

```bash
# Ensure Jaeger is running
docker run -d --name jaeger -p 4318:4318 -p 16686:16686 jaegertracing/all-in-one:1.57

# Run simulation
./ns3 run otlp-basic-example
```

---

## Example 2: Wi-Fi Ad-hoc Simulation (`examples/otlp-wifi-example.cc`)

### Code Overview

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "ns3/applications-module.h"
#include "ns3/core-module.h"
#include "ns3/internet-module.h"
#include "ns3/mobility-module.h"
#include "ns3/network-module.h"
#include "ns3/otlp-module.h"
#include "ns3/wifi-module.h"

using namespace ns3;

int main(int argc, char* argv[])
{
    CommandLine cmd(__FILE__);
    cmd.Parse(argc, argv);

    Time::SetResolution(Time::NS);

    OtelHelper otel;
    otel.SetEndpoint("http://localhost:4318/v1/traces");
    otel.SetServiceName("ns3-wifi-adhoc-simulation");
    otel.Install();

    NodeContainer wifiNodes;
    wifiNodes.Create(3);

    YansWifiChannelHelper channel = YansWifiChannelHelper::Default();
    YansWifiPhyHelper phy;
    phy.SetChannel(channel.Create());

    WifiMacHelper mac;
    WifiHelper wifi;
    wifi.SetStandard(WIFI_STANDARD_80211b);
    wifi.SetRemoteStationManager("ns3::ConstantRateWifiManager");
    mac.SetType("ns3::AdhocWifiMac");

    NetDeviceContainer wifiDevices = wifi.Install(phy, mac, wifiNodes);

    MobilityHelper mobility;
    Ptr<ListPositionAllocator> positionAlloc = CreateObject<ListPositionAllocator>();
    positionAlloc->Add(Vector(0.0, 0.0, 0.0));
    positionAlloc->Add(Vector(50.0, 0.0, 0.0));
    positionAlloc->Add(Vector(100.0, 0.0, 0.0));
    mobility.SetPositionAllocator(positionAlloc);
    mobility.SetMobilityModel("ns3::ConstantPositionMobilityModel");
    mobility.Install(wifiNodes);

    InternetStackHelper stack;
    stack.Install(wifiNodes);

    Ipv4AddressHelper address;
    address.SetBase("192.168.1.0", "255.255.255.0");
    Ipv4InterfaceContainer interfaces = address.Assign(wifiDevices);

    uint16_t port = 9;
    UdpEchoServerHelper echoServer(port);
    ApplicationContainer serverApps = echoServer.Install(wifiNodes.Get(2));
    serverApps.Start(Seconds(1.0));
    serverApps.Stop(Seconds(11.0));

    UdpEchoClientHelper echoClient(interfaces.GetAddress(2), port);
    echoClient.SetAttribute("MaxPackets", UintegerValue(15));
    echoClient.SetAttribute("Interval", TimeValue(Seconds(0.5)));
    echoClient.SetAttribute("PacketSize", UintegerValue(1024));

    ApplicationContainer clientApps = echoClient.Install(wifiNodes.Get(0));
    clientApps.Start(Seconds(2.0));
    clientApps.Stop(Seconds(11.0));

    otel.EnableNodeTracing(wifiNodes);

    Simulator::Stop(Seconds(12.0));
    Simulator::Run();
    Simulator::Destroy();

    return 0;
}
```

### Running the Example

```bash
./ns3 run otlp-wifi-example
```
