# 09. Automated Unit Testing

This guide covers running unit tests for `ns3-otlp` using the ns-3 test runner.

---

## 1. Unit Test Architecture

The `otlp-test-suite` uses `InMemorySpanExporter` (`opentelemetry_exporter_in_memory`) from the OpenTelemetry C++ SDK:

- **Isolated Execution:** Tests run completely offline with zero network calls to localhost:4318.
- **Fast CI Execution:** Prevents timeouts or connection stalls when running in headless CI environments.
- **Exact Assertions:** Validates span names, attributes, `OtelClock` conversion accuracy, and error statuses directly in memory.

---

## 2. Running Unit Tests

Execute the OTLP test suite via `./ns3`:

```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 run "test-runner --suite=otlp --verbose"
```

### Expected Output

```
PASS: OtelTraceSinkTestCase
PASS: OtelClockTestCase
1 of 1 test suites passed (1 of 1 tests passed)
```

---

## 3. Test Suite Implementation (`test/otlp-test-suite.cc`)

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "ns3/log.h"
#include "ns3/otlp-clock-provider.h"
#include "ns3/otlp-trace-sink.h"
#include "ns3/packet.h"
#include "ns3/test.h"

#include <opentelemetry/exporter/memory/in_memory_span_exporter.h>
#include <opentelemetry/exporter/memory/in_memory_span_data.h>
#include <opentelemetry/sdk/trace/simple_processor_factory.h>
#include <opentelemetry/sdk/trace/tracer_provider_factory.h>
#include <opentelemetry/trace/provider.h>

using namespace ns3;
using opentelemetry::exporter::memory::SpanData;

class OtelTraceSinkTestCase : public TestCase
{
  public:
    OtelTraceSinkTestCase() : TestCase("Test OtelTraceSink using InMemorySpanExporter") {}

    void DoRun() override
    {
        const size_t maxSpans = 100;
        auto data = std::make_shared<SpanData>(maxSpans);
        auto exporter = opentelemetry::exporter::memory::InMemorySpanExporterFactory::Create(data);
        auto processor = opentelemetry::sdk::trace::SimpleSpanProcessorFactory::Create(std::move(exporter));
        auto sdkProvider = opentelemetry::sdk::trace::TracerProviderFactory::Create(std::move(processor));

        opentelemetry::nostd::shared_ptr<opentelemetry::trace::TracerProvider> provider(sdkProvider.release());
        opentelemetry::trace::Provider::SetTracerProvider(provider);

        Ptr<OtelTraceSink> sink = CreateObject<OtelTraceSink>();
        sink->InitTracer();

        Ptr<Packet> p = Create<Packet>(512);
        sink->TracePacketTx(p, 1);
        sink->TracePacketRx(p, 2);
        sink->TracePacketDrop(p, 1, "BUFFER_FULL");

        auto spans = data->GetSpans();
        NS_TEST_ASSERT_MSG_EQ(spans.size(), 3, "Should have exported 3 spans");
        NS_TEST_ASSERT_MSG_EQ(spans[0]->GetName(), "packet_tx", "Span 0 should be packet_tx");
        NS_TEST_ASSERT_MSG_EQ(spans[1]->GetName(), "packet_rx", "Span 1 should be packet_rx");
        NS_TEST_ASSERT_MSG_EQ(spans[2]->GetName(), "packet_drop", "Span 2 should be packet_drop");
    }
};
```
