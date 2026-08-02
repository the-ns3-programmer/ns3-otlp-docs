# 10. Troubleshooting & FAQ

This page addresses common compilation, linking, and runtime issues.

---

## Common Errors & Fixes

### 1. Linker Relocation Error (`R_X86_64_TPOFF32`)
**Symptom:**
```text
/usr/bin/ld: /usr/local/lib/libopentelemetry_trace.a: relocation R_X86_64_TPOFF32 against symbol ... can not be used when making a shared object; recompile with -fPIC
```
**Cause:** `opentelemetry-cpp` was compiled as static libraries without position-independent code.
**Fix:** Rebuild `opentelemetry-cpp` with `-DCMAKE_POSITION_INDEPENDENT_CODE=ON` and `-DBUILD_SHARED_LIBS=ON`.

### 2. Missing Header `clock.h`
**Symptom:** `fatal error: opentelemetry/sdk/common/clock/clock.h: No such file or directory`
**Cause:** Include path `/usr/local/include` missing from target or incorrect header name.
**Fix:** `target_include_directories(otlp PUBLIC /usr/local/include ${OPENTELEMETRY_INCLUDE_DIRS})` and use `#include <opentelemetry/common/timestamp.h>`.

### 3. Docker Container Network Orphan Error
**Symptom:** `ERROR: error while removing network: network arun-dot-com_default has active endpoints`
**Fix:** `docker-compose down --remove-orphans`

---

## BibTeX Citation

```bibtex
@software{ns3_otlp_2026,
  author = {Arun Santhosh R A},
  title = {ns3-otlp: Native OpenTelemetry Exporter Module for Network Simulator 3},
  year = {2026},
  version = {1.0.0},
  url = {https://github.com/the-ns3-programmer/ns3-otlp}
}
```
