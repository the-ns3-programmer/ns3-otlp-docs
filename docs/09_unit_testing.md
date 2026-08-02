# 09. Automated Unit Testing Framework

## Overview

`ns3-otlp` includes an automated unit test suite (`test/otlp-test-suite.cc`) integrated directly into ns-3's `test-runner`.

---

## Test Suite Implementation (`contrib/otlp/test/otlp-test-suite.cc`)

```cpp
#include "ns3/test.h"
#include "ns3/otlp-module.h"
#include "ns3/simulator.h"
#include "ns3/packet.h"

namespace ns3 {

// 1. Clock Test Case
class OtelClockTestCase : public TestCase
{
public:
  OtelClockTestCase();
  ~OtelClockTestCase() override = default;

private:
  void DoRun() override;
};

OtelClockTestCase::OtelClockTestCase()
  : TestCase("Check OtelClock timestamp conversion with Simulator::Now()")
{
}

void
OtelClockTestCase::DoRun()
{
  Time simTime = NanoSeconds(100);
  Simulator::Stop(simTime);
  Simulator::Run();

  auto otelTime = OtelClock::GetNow();
  NS_TEST_ASSERT_MSG_EQ(Simulator::Now().GetNanoSeconds(), 100, "Simulator time should be 100 ns");
  NS_TEST_ASSERT_MSG_GT(otelTime.time_since_epoch().count(), 0, "OtelClock timestamp should be > 0");

  Simulator::Destroy();
}

// 2. Trace Sink Test Case
class OtelTraceSinkTestCase : public TestCase
{
public:
  OtelTraceSinkTestCase();
  ~OtelTraceSinkTestCase() override = default;

private:
  void DoRun() override;
};

OtelTraceSinkTestCase::OtelTraceSinkTestCase()
  : TestCase("Check OtelTraceSink packet event handling")
{
}

void
OtelTraceSinkTestCase::DoRun()
{
  OtelHelper otel("ns3-unit-test");
  auto traceSink = otel.GetTraceSink();

  Ptr<Packet> p = Create<Packet>(512);
  NS_TEST_ASSERT_MSG_NE(p, nullptr, "Packet creation failed");

  traceSink->TracePacketTx(p, 0);
  traceSink->TracePacketRx(p, 1);
  traceSink->TracePacketDrop(p, 0, "TEST_DROP");

  Simulator::Destroy();
}

// 3. Test Suite Class
class OtlpTestSuite : public TestSuite
{
public:
  OtlpTestSuite();
};

OtlpTestSuite::OtlpTestSuite()
  : TestSuite("otlp", TestSuite::Type::UNIT)
{
  AddTestCase(new OtelClockTestCase, TestCase::Duration::QUICK);
  AddTestCase(new OtelTraceSinkTestCase, TestCase::Duration::QUICK);
}

static OtlpTestSuite g_otlpTestSuite;

} // namespace ns3
```

---

## Executing Unit Tests

```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 run "test-runner --suite=otlp"
```

Expected output:
```text
PASS otlp 0.021 s
```
