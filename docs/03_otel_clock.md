# 03. Component 1: OtelClock (Virtual Time Synchronization)

## Purpose

Standard OpenTelemetry SDK derives timestamps from host wall-clock time (`std::chrono::system_clock::now()`). In discrete-event simulation, events execute non-linearly with respect to real wall-clock time. A 10-second simulation might execute in 50ms of CPU time or 5 minutes.

`OtelClock` provides virtual simulation clock conversion routines mapping `ns3::Simulator::Now()` nanoseconds directly into `opentelemetry::common::SystemTimestamp`.

---

## Class Definition (`model/otlp-clock-provider.h`)

```cpp
#ifndef OTLP_CLOCK_PROVIDER_H
#define OTLP_CLOCK_PROVIDER_H

#include "ns3/nstime.h"
#include "ns3/simulator.h"

#include <opentelemetry/common/timestamp.h>

namespace ns3 {

/**
 * \ingroup otlp
 * \brief Utility to convert ns-3 virtual simulation time into OpenTelemetry timestamps.
 */
class OtelClock
{
public:
  /**
   * \return Current simulation time as OpenTelemetry SystemTimestamp.
   */
  static opentelemetry::common::SystemTimestamp GetNow() noexcept;

  /**
   * \return Given ns-3 Time as OpenTelemetry SystemTimestamp.
   */
  static opentelemetry::common::SystemTimestamp ToSystemTimestamp(Time t) noexcept;
};

} // namespace ns3

#endif // OTLP_CLOCK_PROVIDER_H
```

---

## Implementation (`model/otlp-clock-provider.cc`)

```cpp
#include "otlp-clock-provider.h"

namespace ns3 {

opentelemetry::common::SystemTimestamp
OtelClock::GetNow() noexcept
{
  return opentelemetry::common::SystemTimestamp(
      std::chrono::nanoseconds(Simulator::Now().GetNanoSeconds()));
}

opentelemetry::common::SystemTimestamp
OtelClock::ToSystemTimestamp(Time t) noexcept
{
  return opentelemetry::common::SystemTimestamp(
      std::chrono::nanoseconds(t.GetNanoSeconds()));
}

} // namespace ns3
```

---

## Technical Rationale

1. **Precision:** Reads nanoseconds via `Simulator::Now().GetNanoSeconds()` to preserve microsecond/nanosecond network event granularity.
2. **Compatibility:** Returns standard `opentelemetry::common::SystemTimestamp` objects accepted by `opentelemetry::trace::StartSpanOptions::start_system_time`.
