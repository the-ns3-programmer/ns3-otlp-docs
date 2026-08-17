# 03. Component 1: OtelClock (Virtual Time Synchronization)

## Overview & Purpose

Standard OpenTelemetry SDKs derive timestamps from host wall-clock time (`std::chrono::system_clock::now()`). In discrete-event simulation, relative simulation time starts at 0 ns (`Simulator::Now() = 0`). If nanoseconds are converted directly into system timestamps, spans are dated January 1, 1970, which Jaeger UI and modern observability collectors reject or hide outside the active lookback window.

`OtelClock` provides virtual simulation clock conversion routines mapping `ns3::Simulator::Now()` onto a wall-clock baseline captured at `Install()` time (`wall_base + Simulator::Now()`).

---

## Detailed API Specification

Header: `ns3/otlp-clock-provider.h`  
Namespace: `ns3`

### Method Summary

```cpp
namespace ns3 {

class OtelClock
{
public:
    /**
     * \brief Set the wall-clock baseline time point.
     * \param base System time point captured when OtelHelper::Install() is called.
     */
    static void SetWallClockBase(std::chrono::system_clock::time_point base) noexcept;

    /**
     * \return Current simulation time anchored to wall-clock baseline.
     */
    static opentelemetry::common::SystemTimestamp GetNow() noexcept;

    /**
     * \param t ns-3 Time object.
     * \return Given ns-3 Time anchored to wall-clock baseline.
     */
    static opentelemetry::common::SystemTimestamp ToSystemTimestamp(Time t) noexcept;
};

} // namespace ns3
```

### Detailed Method Descriptions

#### `SetWallClockBase(std::chrono::system_clock::time_point base)`
- **Parameters:** `base` — The `std::chrono::system_clock::time_point` captured at simulation initialization.
- **Description:** Sets the baseline real-world timestamp. Called automatically by `OtelHelper::Install()`.
- **Thread Safety:** Thread-safe for read operations after initialization.

#### `GetNow()`
- **Returns:** `opentelemetry::common::SystemTimestamp` representing $\text{wall\_base} + \text{Simulator::Now()}$.
- **Description:** Converts the current relative simulation time (`Simulator::Now()`) into a absolute `SystemTimestamp` compatible with OpenTelemetry `StartSpanOptions`.

#### `ToSystemTimestamp(Time t)`
- **Parameters:** `t` — An ns-3 `Time` instance.
- **Returns:** `opentelemetry::common::SystemTimestamp` representing $\text{wall\_base} + t$.
- **Description:** Converts any relative ns-3 simulation timestamp into a wall-clock anchored `SystemTimestamp`.

---

## Implementation Code (`model/otlp-clock-provider.cc`)

```cpp
// SPDX-License-Identifier: GPL-3.0-or-later
// Copyright (C) 2026 Arun Santhosh R A <arunsanthosh.rashok@gmail.com>

#include "otlp-clock-provider.h"

namespace ns3
{

static std::chrono::system_clock::time_point g_wallClockBase = std::chrono::system_clock::now();

void
OtelClock::SetWallClockBase(std::chrono::system_clock::time_point base) noexcept
{
    g_wallClockBase = base;
}

opentelemetry::common::SystemTimestamp
OtelClock::GetNow() noexcept
{
    auto ns = std::chrono::nanoseconds(Simulator::Now().GetNanoSeconds());
    return opentelemetry::common::SystemTimestamp(g_wallClockBase + ns);
}

opentelemetry::common::SystemTimestamp
OtelClock::ToSystemTimestamp(Time t) noexcept
{
    auto ns = std::chrono::nanoseconds(t.GetNanoSeconds());
    return opentelemetry::common::SystemTimestamp(g_wallClockBase + ns);
}

} // namespace ns3
```
