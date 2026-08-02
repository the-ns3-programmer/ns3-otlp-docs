# 06. Component 4: OtelHelper (Initialization & Exporter Pipeline)

## Purpose

`OtelHelper` is the central user-facing API component. Following ns-3 helper conventions (`InternetStackHelper`, `FlowMonitorHelper`), it configures the OpenTelemetry export pipeline, sets up HTTP/gRPC exporters, instantiates processors, and binds sinks to simulation nodes.

---

## Class Definition (`helper/otlp-helper.h`)

```cpp
#ifndef OTLP_HELPER_H
#define OTLP_HELPER_H

#include "ns3/node-container.h"
#include "ns3/net-device-container.h"
#include "ns3/otlp-trace-sink.h"
#include "ns3/otlp-metric-sink.h"

#include <string>
#include <memory>

namespace ns3 {

/**
 * \ingroup otlp
 * \brief User-facing helper to configure OpenTelemetry OTLP export and trace hooks in ns-3.
 */
class OtelHelper
{
public:
  OtelHelper(const std::string &serviceName = "ns3-simulation");
  ~OtelHelper() = default;

  /**
   * \brief Set the OTLP collector endpoint (e.g. "http://localhost:4318/v1/traces").
   */
  void SetEndpoint(const std::string &endpoint);

  /**
   * \brief Set the service name for OpenTelemetry resource.
   */
  void SetServiceName(const std::string &serviceName);

  /**
   * \brief Configure and initialize the OTLP Exporter and Tracer Providers.
   */
  void ConfigureExporter();

  /**
   * \brief Enable packet trace hooks on a single node or node container.
   */
  void EnableNodeTracing(Ptr<Node> node);
  void EnableNodeTracing(NodeContainer nodes);

  /**
   * \brief Get the underlying OtelTraceSink.
   */
  std::shared_ptr<OtelTraceSink> GetTraceSink() const;

  /**
   * \brief Get the underlying OtelMetricSink.
   */
  std::shared_ptr<OtelMetricSink> GetMetricSink() const;

private:
  std::string m_endpoint{"http://localhost:4318/v1/traces"};
  std::string m_serviceName{"ns3-simulation"};
  bool m_configured{false};

  std::shared_ptr<OtelTraceSink> m_traceSink;
  std::shared_ptr<OtelMetricSink> m_metricSink;
};

} // namespace ns3

#endif // OTLP_HELPER_H
```

---

## Implementation (`helper/otlp-helper.cc`)

```cpp
#include "otlp-helper.h"
#include "ns3/log.h"
#include "ns3/config.h"

#include <opentelemetry/exporters/otlp/otlp_http_exporter_factory.h>
#include <opentelemetry/exporters/otlp/otlp_http_exporter_options.h>
#include <opentelemetry/sdk/trace/tracer_provider_factory.h>
#include <opentelemetry/sdk/trace/simple_processor_factory.h>
#include <opentelemetry/trace/provider.h>
#include <opentelemetry/sdk/resource/resource.h>

namespace ns3 {

NS_LOG_COMPONENT_DEFINE("OtelHelper");

OtelHelper::OtelHelper(const std::string &serviceName)
  : m_serviceName(serviceName)
{
  ConfigureExporter();
  m_traceSink = std::make_shared<OtelTraceSink>();
  m_metricSink = std::make_shared<OtelMetricSink>();
}

void
OtelHelper::SetEndpoint(const std::string &endpoint)
{
  m_endpoint = endpoint;
}

void
OtelHelper::SetServiceName(const std::string &serviceName)
{
  m_serviceName = serviceName;
}

void
OtelHelper::ConfigureExporter()
{
  if (m_configured) return;

  opentelemetry::exporter::otlp::OtlpHttpExporterOptions opts;
  opts.url = m_endpoint;

  auto exporter = opentelemetry::exporter::otlp::OtlpHttpExporterFactory::Create(opts);

  auto processor = opentelemetry::sdk::trace::SimpleSpanProcessorFactory::Create(
      std::move(exporter));

  opentelemetry::sdk::resource::ResourceAttributes attributes = {
      {"service.name", m_serviceName}};
  auto resource = opentelemetry::sdk::resource::Resource::Create(attributes);

  auto tracerProviderUnique = opentelemetry::sdk::trace::TracerProviderFactory::Create(
      std::move(processor), resource);

  std::shared_ptr<opentelemetry::trace::TracerProvider> sdkProvider = std::move(tracerProviderUnique);
  opentelemetry::nostd::shared_ptr<opentelemetry::trace::TracerProvider> provider(sdkProvider);

  opentelemetry::trace::Provider::SetTracerProvider(provider);
  m_configured = true;
}

void
OtelHelper::EnableNodeTracing(Ptr<Node> node)
{
  if (!node) return;
  ConfigureExporter();
}

void
OtelHelper::EnableNodeTracing(NodeContainer nodes)
{
  for (auto it = nodes.Begin(); it != nodes.End(); ++it)
  {
    EnableNodeTracing(*it);
  }
}

std::shared_ptr<OtelTraceSink>
OtelHelper::GetTraceSink() const
{
  return m_traceSink;
}

std::shared_ptr<OtelMetricSink>
OtelHelper::GetMetricSink() const
{
  return m_metricSink;
}

} // namespace ns3
```
