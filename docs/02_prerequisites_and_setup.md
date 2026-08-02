# 02. Prerequisites & Environment Setup

This guide details the complete prerequisite stack, system dependencies, library setup, and CMake build configuration required to build `ns3-otlp` on Linux.

---

## 1. System Requirements

- **Supported OS:** Ubuntu 22.04 LTS / 24.04 LTS / Debian 12 / WSL2 Linux
- **C++ Compiler:** GCC 11+ or Clang 13+ with C++17/C++20/C++23 support
- **Build System:** CMake 3.22+ and Ninja build
- **ns-3 Version:** ns-3.36 or newer (tested on ns-3.46.1)

---

## 2. Installing System Packages

```bash
sudo apt update && sudo apt install -y \
    build-essential \
    g++ \
    gdb \
    cmake \
    ninja-build \
    pkg-config \
    git \
    python3 \
    python3-pip \
    ccache \
    libcurl4-openssl-dev \
    libssl-dev \
    zlib1g-dev \
    protobuf-compiler \
    libprotobuf-dev \
    libgrpc++-dev \
    protobuf-compiler-grpc
```

---

## 3. Building & Installing `opentelemetry-cpp` SDK

The OpenTelemetry C++ SDK must be built as **shared libraries** (`-DBUILD_SHARED_LIBS=ON`) with **Position Independent Code** (`-DCMAKE_POSITION_INDEPENDENT_CODE=ON`) so that ns-3 can link it into shared libraries (`libns3-otlp-default.so`).

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
      -DWITH_OTLP_GRPC=ON \
      -DWITH_OTLP_HTTP=ON ..

make -j$(nproc)
sudo make install
sudo ldconfig
```

---

## 4. Configuring ns-3 Build System

To enable `ns3-otlp` inside ns-3:

```bash
cd ~/ns-allinone-3.46.1/ns-3.46.1
./ns3 configure --enable-examples --enable-tests
./ns3 build otlp
```

### Verification
Check that `pkg-config` detects OpenTelemetry:

```bash
pkg-config --modversion opentelemetry_trace
# Expected output: 1.29.0-dev (or installed version)
```
