# 02. Prerequisites & Environment Setup

This guide details the system prerequisites, library setup, and CMake build configuration required to build `ns3-otlp` on Linux.

---

## 1. System Requirements

- **Supported OS:** Ubuntu 22.04 LTS / 24.04 LTS / Debian 12 / WSL2 Linux
- **C++ Compiler:** GCC 11+ or Clang 13+ with C++17/C++20/C++23 support
- **Build System:** CMake 3.22+ and Ninja build
- **ns-3 Version:** ns-3.36 or newer (tested on ns-3.46.1)
- **Containers:** Docker (for running Jaeger UI container)

---

## 2. Installing System Packages

```bash
sudo apt update && sudo apt install -y \
    build-essential \
    g++ \
    cmake \
    ninja-build \
    pkg-config \
    git \
    libcurl4-openssl-dev \
    libssl-dev \
    zlib1g-dev \
    protobuf-compiler \
    libprotobuf-dev \
    libgrpc++-dev
```

---

## 3. Building & Installing `opentelemetry-cpp` SDK

The OpenTelemetry C++ SDK must be built with shared libraries (`-DBUILD_SHARED_LIBS=ON`), position-independent code (`-DCMAKE_POSITION_INDEPENDENT_CODE=ON`), and OTLP HTTP exporter enabled:

```bash
cd ~
git clone --recursive https://github.com/open-telemetry/opentelemetry-cpp.git
cd opentelemetry-cpp
mkdir build && cd build

cmake -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DBUILD_TESTING=OFF \
      -DWITH_BENCHMARK=OFF \
      -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
      -DBUILD_SHARED_LIBS=ON \
      -DWITH_OTLP_GRPC=OFF \
      -DWITH_OTLP_HTTP=ON \
      -DWITH_METRICS_EXEMPLAR_PREVIEW=ON ..

make -j$(nproc)
sudo make install
sudo ldconfig
```

---

## 4. Configuring ns-3 Build System

To build `ns3-otlp` inside ns-3:

```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 configure --enable-examples --enable-tests
./ns3 build otlp
```

### Graceful Fallback Verification
If `opentelemetry-cpp` is absent, `./ns3 configure` outputs:
```
ns3-otlp: opentelemetry-cpp not found via pkg-config — skipping module.
```
and completes without error.
