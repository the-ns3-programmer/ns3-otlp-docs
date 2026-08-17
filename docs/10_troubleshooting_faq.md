# 10. Troubleshooting & FAQ

This guide resolves common build errors, runtime issues, and configuration problems when using `ns3-otlp`.

---

## 1. Common Issues & Solutions

### Issue 1: `opentelemetry-cpp not found via pkg-config`
- **Symptom:** `./ns3 configure` prints `ns3-otlp: opentelemetry-cpp not found via pkg-config — skipping module.`
- **Cause:** `opentelemetry-cpp` shared libraries or `.pc` files are missing from standard library search paths.
- **Solution:**
  1. Ensure `opentelemetry-cpp` was installed with `-DCMAKE_INSTALL_PREFIX=/usr/local`.
  2. Verify PKG_CONFIG_PATH contains `/usr/local/lib/pkgconfig`:
     ```bash
     export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
     pkg-config --modversion opentelemetry_trace
     ```

### Issue 2: Docker Compose legacy error (`ContainerConfig` KeyError)
- **Symptom:** `docker-compose up` fails with `KeyError: 'ContainerConfig'` when using old `docker-compose 1.29.2`.
- **Cause:** Standalone Docker Compose v1 is incompatible with modern Docker engine API updates.
- **Solution:** Use direct `docker run` command:
  ```bash
  docker run -d --name jaeger \
    -p 4318:4318 \
    -p 16686:16686 \
    jaegertracing/all-in-one:1.57
  ```

### Issue 3: Spans not appearing in Jaeger UI
- **Symptom:** Simulation runs without error, but no traces appear when searching service in Jaeger.
- **Cause:** `OtelHelper::Install()` or `OtelHelper::EnableNodeTracing()` was not called.
- **Solution:** Ensure sequence in `main()` is:
  ```cpp
  OtelHelper otel;
  otel.SetServiceName("my-service");
  otel.Install();
  // ... create nodes & devices ...
  otel.EnableNodeTracing(nodes);
  ```

### Issue 4: `target_link_libraries` signature mismatch error during CMake configure
- **Symptom:** `The plain signature for target_link_libraries has already been used with the target "otlp-test".`
- **Cause:** Mixing CMake `PRIVATE` keyword with ns-3's internal plain `target_link_libraries` calls.
- **Solution:** Maintain plain `target_link_libraries(otlp-test ${libotlp} ${OPENTELEMETRY_LIBRARIES})` in `CMakeLists.txt`.

---

## 2. Frequently Asked Questions (FAQ)

#### Q: What C++ standard does `ns3-otlp` require?
`ns3-otlp` supports C++17, C++20, and C++23.

#### Q: Does `ns3-otlp` affect simulation execution speed?
Trace and metric data are processed asynchronously via OpenTelemetry SDK batch span processors, minimizing overhead during event execution.

#### Q: What license is `ns3-otlp` released under?
GNU General Public License v3.0 or later (`GPL-3.0-or-later`), making it compatible with Apache-2.0 third-party libraries (`opentelemetry-cpp` and `gRPC`).
