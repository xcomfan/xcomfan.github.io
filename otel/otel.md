---
layout: page
title: "Open Telemetry"
permalink: /otel
---

## Working My Way Through The OTEL demo

### Questions

[ ] What is gRPC and why are we using it here?

## Ad-hoc Notes

* **MELT** = **M**etrics **E**vents **L**ogs and **T**races

* Span context consists of the below in addition to info needed to identify current span and trace.
  * Trace ID
  * Span ID
  * Trace Flags
  * Trace State

* Correlation Context is optional and is made up of things such as ...
  * Customer ID
  * Host Name
  * Region
  * Other application specific performance insights

* Propagation is the mechanism used to bundle up the context and transfer it among services.

* Combined propagation nd context combine the engine behind distributed tracing.

## FreeCodeCamp Course Notes

[Course Link](https://youtu.be/r8UvWSX3KA8?si=elB3iPN508vZP-CQ)

## Documentation Notes

PLACEHOLDER

## Udemy Course Notes

Main differences between monitoring and observability are ...

* Monitoring can do metrics and logs but observability can also do tracing.
* Monitoring uses static threshold but observability can do automatic thresholds.
* Observability can help you find the underlying cause of an alert while monitoring will just alert.

Otel collects ...

* Metrics - Point in time values
* Logs
* Traces
* Baggage

Collector is a proxy server that collects the data, processes it and sends it to a backend of your choice.

### Metrics

If you are going to collect metrics in Otel you need to initialize a **Meter Provider**. This is typically done once globally and the application uses this single Meter Provider to create Meters (Thing of the Meter Provider as a Meter Factory). The **Metric Exporter** send the metrics to either a collector or a back end.

Otel has support for the following metrics types

* Counter - only increases (get reset when application is restarted)

* Asynchronous Counter - Like a counter, but collected at regular intervals instead of being directly emitted by application code. Collected once per period (once every 2 minutes for example)

* UpDownCounter - Can be incremented or decremented. Useful for tracking numbers that go up and down such as current memory usage or number of active connections.

* Async UpDownCounter - Like UpDownCounter but updated asynchronously. Useful for values that fluctuate over time and need to be sampled at regular intervals.

* Gauge - Unlike counters are not cumulative. Can be used for current CPU/Memory usage.

* Histogram - Measures observations and counts them in configurable buckets. Used for understanding performance characteristics of your application.

In the demo application the monitor/observability solutions used are

* Prometheus for metrics
* Jaeger for traces
* OpenSearch for logs

### Traces

Just like a Metric Providers, Traces need a **Tracer** that is generated from a **Tracer Provider** which is like a factory for Tracers. The Tracers then generate spans which once generated are exported using the **Trace Exporter**.

Span is a unit for work and there are 9 pieces of information collected in a Span.

* Name
* Span Context
  * Trace ID: Full request path id
  * Span ID: this span's id
  * Trace Flags: Sampled?
  * Trace State: Custom vendor-info
* Parent Span ID
* Start and End Timestamps
* Attributes
  * Server Info
  * Process Info
  * OTel Info
  * Custom Info
* Span Events - Any info you want to pass with the span
* Span Links - Associate spans in async transactions
* Span Status - Values can be unset, error or OK. Unset means completed successfully. OK means completed successfully but the status is set explicitly by a developer.
* Span Kind - Values can be client, server, internal, producer, consumer

## Collector Configuration

The `config.yaml` file has the following structure 

```text
receivers:

processors:

extensions:

service:
  extensions:
  pipelines:
  metrics:
  logs:
  traces:
```

We will dive into these sections below...

### receivers

Responsible for receiving telemetry data which comes from instrumentation. Some receivers are capable of fetching data instead of listening for it. Some example receivers are OTLP, Prometheus, Jaeger.

### processors

Processors modify the received data. They can add/modify/remove data from collected/received data based on the processor settings. For example batch processor batches the data into batches to help compress it and reduce the number of connections. Some examples of processors are Batch, Memory Limiter, Attributes, Cumulative to Delta, Metrics Generation. Processors can receive data from one or more pipelines.

### exporters

Exporters send the process data to a desired location. This can be directly to an observability back end, another collector (if its a Gateway collector), or to a log file. Most exporters require some level of configuration. Some examples of exporters are Debug (send data to standard out), OTLP gRPC, OTLP HTTP, Prometheus, Jaeger, Elasticsearch. Also can receive data from multiple pipelines.

### extensions

Used by the collector to collect additional data. Allow you to expand what the collector does. For example health_check extension will add an HTTP endpoint that allows you to check status of the collector. The `pprof` extension allows for performance profiling of the collector. `zpages` extension provides in process page that allows you to see tracing debug data.

### services

Enables components from the receivers, processors and exporters sections. If its listed in the other sections, but not in the services section it is not enabled.

#### extensions(service)

#### pipelines

Connects the receivers, processors and exporters into a pipeline. For example is you have receiver 1 and 2 metrics received from receiver 1 can be sent to processor 1 and exported via exporter 2. We can build pipelines for metrics, logs and traces.

Here is a sample configuration. Where trace data comes though otlp, opencensus, jaeger, and zipkin. Gets batched and exported to a debug log file.

```yaml
services:
  pipelines:
    traces:
      receivers: [otlp, opencensus, jaeger, zipkin]
      processors: [batch]
      exporters: [debug]

    metrics:
      receivers: [otlp, opencensus, prometheus]
      processors: [batch]
      exporters: [debug]

    logs:
      receivers: [otlp]
      processors: [batch]
      exporters: [debug]

  extensions: [health_check, pprof, zpages]
```
