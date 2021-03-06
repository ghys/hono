+++
title = "Metrics"
weight = 440
+++

Eclipse Hono&trade;'s components report several metrics which may be used to gain some insight
into the running system. For instance, the HTTP adapter reports the number of successfully
processed telemetry messages. Some of these metrics are considered part of Hono's external
interface. This section describes the semantics and format of the metrics, how they can be retrieved
and how to interpret actual values.
<!--more-->

## Reported Metrics

Hono uses [Micrometer](https://micrometer.io/) in combination with Spring Boot
to internally collect metrics. Those metrics can be exported to different
back ends. Please refer to [Configuring Metrics]({{< ref "/admin-guide/monitoring-tracing-config.md#configuring-metrics" >}})
for details.

The example deployment by default uses [Prometheus](https://prometheus.io/) as the metrics back end.

When deploying to Kubernetes/OpenShift, the metrics reported by Hono may contain
environment specific tags (like the *pod* name) which are added by the Prometheus
scraper. However, those tags are not part of the Hono metrics definition.

Hono applications may report other metrics in addition to the ones defined here.
In particular, all components report metrics regarding the JVM's internal state, e.g.
memory consumption and garbage collection status. Those metrics are not considered
part of Hono's *official* metrics definition. However, all those metrics
will still contain tags as described below.

### Common Metrics

Tags for common metrics are:

| Tag              | Value                              | Description |
| ---------------- | ---------------------------------- | ----------- |
| *host*           | *string*                           | The name of the host that the component reporting the metric is running on |
| *component-type* | `adapter`, `service`             | The type of component reporting the metric |
| *component-name* | *string*                           | The name of the component reporting the metric. |

The names of Hono's standard components are as follows:

| Component         | *component-name*      |
| ----------------- | --------------------- |
| Auth Server       | `hono-auth`         |
| Device Registry   | `hono-registry`     |
| Hono Messaging    | `hono-messaging`    |
| AMQP adapter      | `hono-amqp`         |
| CoAP adapter      | `hono-coap`         |
| HTTP adapter      | `hono-http`         |
| Kura adapter      | `hono-kura-mqtt`   |
| MQTT adapter      | `hono-mqtt`         |

### Protocol Adapter Metrics

Additional tags for protocol adapters are:

| Tag        | Value                              | Description |
| ---------- | ---------------------------------- | ----------- |
| *tenant*   | *string*                           | The name of the tenant that the metric is being reported for |
| *type*     | `telemetry`, `event`             | The type of message that the metric is being reported for |

Metrics provided by the protocol adapters are:

| Metric                             | Type    | Tags                                                         | Description |
| ---------------------------------- | ------- | ------------------------------------------------------------ | ----------- |
| *hono.connections.authenticated*   | Gauge   | *host*, *component-type*, *component-name*, *tenant*         | Current number of connected, authenticated devices. <br/> **NB** This metric is only supported by protocol adapters that maintain *connection state* with authenticated devices. In particular, the HTTP adapter does not support this metric. |
| *hono.connections.unauthenticated* | Gauge   | *host*, *component-type*, *component-name*                   | Current number of connected, unauthenticated devices. <br/> **NB** This metric is only supported by protocol adapters that maintain *connection state* with authenticated devices. In particular, the HTTP adapter does not support this metric. |
| *hono.messages.undeliverable*      | Counter | *host*, *component-type*, *component-name*, *tenant*, *type* | Total number of undeliverable messages |
| *hono.messages.processed*          | Counter | *host*, *component-type*, *component-name*, *tenant*, *type* | Total number of processed messages |
| *hono.messages.processed.payload*  | Counter | *host*, *component-type*, *component-name*, *tenant*, *type* | Total number of processed payload bytes |
| *hono.commands.device.delivered*   | Counter | *host*, *component-type*, *component-name*, *tenant*         | Total number of delivered commands |
| *hono.commands.ttd.expired*        | Counter | *host*, *component-type*, *component-name*, *tenant*         | Total number of expired TTDs |
| *hono.commands.response.delivered* | Counter | *host*, *component-type*, *component-name*, *tenant*         | Total number of delivered responses to commands |

### Service Metrics

Hono's service components do not report any metrics at the moment.

## Legacy Metrics

{{%note title="Deprecated"%}}
This metrics configuration is still supported in Hono, but is considered
deprecated and will be removed in a future version of Hono.

The legacy metrics support needs to be specifically enabled, starting with
Hono 0.9. See [Legacy support](/hono/admin-guide/monitoring-tracing-config#legacy-metrics-support) 
for more information.
{{%/note%}}

Enabling the legacy support sets up internal support for exporting the metrics
in the pre-0.8 format, using the Graphite Micrometer backend, but with a
custom (legacy) naming scheme. It still is possible to use the InfluxDB Micrometer
backend as well, however this will export metrics in the new format, and not
on the legacy format.

Some users have started to build custom dash boards on top of the names of the metrics and
associated tags that end up in the example InfluxDB. Hono will try to support such users
by allowing future versions of Hono to be configured to report metrics to InfluxDB resulting
in the same database structure as created by InfluxDB's Graphite Input.

**NB** This does not necessarily mean that future versions of Hono will always support
metrics being reported using the Graphite wire format. It only means that there will be
a way to transmit metrics to InfluxDB resulting in the same database structure.

### Tags

The InfluxDB configuration file used for the example deployment contains templates for
extracting the following *tags* from metric names transmitted via the Graphite reporter.

| Tag        | Description |
| ---------- | ----------- |
| *host*     | The name of the host that the component reporting the metric is running on. |
| *type*     | The type of message that the metric is being reported for (either `telemetry` or `event`) |
| *tenant*   | The name of the tenant that the metric is being reported for. |
| *protocol* | The transport protocol that has been used to send the message for which the metric is being reported (`http` or `mqtt`). |

The following sections describe which of these tags are extracted for which metrics specifically.

### Common Metrics

The following table contains metrics that are collected for all protocol adapters (if applicable to the underlying transport protocol).

| Metric                                             | Tags                             | Description |
| -------------------------------------------------- | -------------------------------- | ----------- |
| *counter.hono.connections.authenticated.count*     | *host*, *tenant*, *protocol*     | Current number of connections with authenticated devices. **NB** This metric is only supported by protocol adapters that maintain *connection state* with authenticated devices. In particular, the HTTP adapter does not support this metric. |
| *counter.hono.connections.unauthenticated.count*   | *host*, *protocol*               | Current number of connections with unauthenticated devices. **NB** This metric is only supported by protocol adapters that maintain *connection state* with unauthenticated devices. In particular, the HTTP adapter does not support this metric. |
| *meter.hono.commands.device.delivered.count*       | *host*, *tenant*, *protocol*     | Commands delivered to devices. Total count since application start. |
| *meter.hono.commands.device.delivered.m1_rate*     | *host*, *tenant*, *protocol*     | Commands delivered to devices. One minute, exponentially weighted, moving average. |
| *meter.hono.commands.device.delivered.m5_rate*     | *host*, *tenant*, *protocol*     | Commands delivered to devices. Five minute, exponentially weighted, moving average. |
| *meter.hono.commands.device.delivered.m15_rate*    | *host*, *tenant*, *protocol*     | Commands delivered to devices. Fifteen minute, exponentially weighted, moving average. |
| *meter.hono.commands.device.delivered.mean_rate*   | *host*, *tenant*, *protocol*     | Commands delivered to devices. Mean rate of messages since application start. |
| *meter.hono.commands.response.delivered.count*     | *host*, *tenant*, *protocol*     | Command responses delivered to applications. Total count since application startup. |
| *meter.hono.commands.response.delivered.m1_rate*   | *host*, *tenant*, *protocol*     | Command responses delivered to applications. One minute, exponentially weighted, moving average. |
| *meter.hono.commands.response.delivered.m5_rate*   | *host*, *tenant*, *protocol*     | Command responses delivered to applications. Five minute, exponentially weighted, moving average. |
| *meter.hono.commands.response.delivered.m15_rate*  | *host*, *tenant*, *protocol*     | Command responses delivered to applications. Fifteen minute, exponentially weighted, moving average. |
| *meter.hono.commands.response.delivered.mean_rate* | *host*, *tenant*, *protocol*     | Command responses delivered to applications. Mean rate of messages since the application start. |
| *meter.hono.commands.ttd.expired.count*            | *host*, *tenant*, *protocol*     | Messages containing a TTD that expired with no pending command(s). Total count since application startup. |
| *meter.hono.commands.ttd.expired.m1_rate*          | *host*, *tenant*, *protocol*     | Messages containing a TTD that expired with no pending command(s). One minute, exponentially weighted, moving average. |
| *meter.hono.commands.ttd.expired.m5_rate*          | *host*, *tenant*, *protocol*     | Messages containing a TTD that expired with no pending command(s). Five minute, exponentially weighted, moving average. |
| *meter.hono.commands.ttd.expired.m15_rate*         | *host*, *tenant*, *protocol*     | Messages containing a TTD that expired with no pending command(s). Fifteen minute, exponentially weighted, moving average. |
| *meter.hono.commands.ttd.expired.mean_rate*        | *host*, *tenant*, *protocol*     | Messages containing a TTD that expired with no pending command(s). Mean rate of messages since the application start. |

### HTTP Metrics

The following table contains metrics that are collected specifically for the HTTP protocol adapter.

| Metric                                           | Tags                     | Description |
| ------------------------------------------------ | ------------------------ | ----------- |
| *counter.hono.http.messages.undeliverable.count* | *host*, *tenant*, *type* | Messages which could not be processed by the HTTP protocol adapter. Total count since application startup. |
| *meter.hono.http.messages.processed.count*       | *host*, *tenant*, *type* | Messages processed by the HTTP protocol adapter. Total count since application startup. |
| *meter.hono.http.messages.processed.m1_rate*     | *host*, *tenant*, *type* | Messages processed by the HTTP protocol adapter. One minute, exponentially weighted, moving average. |
| *meter.hono.http.messages.processed.m5_rate*     | *host*, *tenant*, *type* | Messages processed by the HTTP protocol adapter. Five minute, exponentially weighted, moving average. |
| *meter.hono.http.messages.processed.m15_rate*    | *host*, *tenant*, *type* | Messages processed by the HTTP protocol adapter. Fifteen minute, exponentially weighted, moving average. |
| *meter.hono.http.messages.processed.mean_rate*   | *host*, *tenant*, *type* | Messages processed by the HTTP protocol adapter. Mean rate of messages since the application start. |

### MQTT Metrics

The following table contains metrics that are collected specifically for the MQTT protocol adapter.

| Metric                                               | Tags                     | Description |
| ---------------------------------------------------- | ------------------------ | ----------- |
| *counter.hono.mqtt.messages.undeliverable.count*     | *host*, *tenant*, *type* | Messages which could not be processed by the MQTT protocol adapter- Total count since application startup. |
| *meter.hono.mqtt.messages.processed.count*           | *host*, *tenant*, *type* | Messages processed by the MQTT protocol adapter. Total count since application startup. |
| *meter.hono.mqtt.messages.processed.m1_rate*         | *host*, *tenant*, *type* | Messages processed by the MQTT protocol adapter. One minute, exponentially weighted, moving average. |
| *meter.hono.mqtt.messages.processed.m5_rate*         | *host*, *tenant*, *type* | Messages processed by the MQTT protocol adapter. Five minute, exponentially weighted, moving average. |
| *meter.hono.mqtt.messages.processed.m15_rate*        | *host*, *tenant*, *type* | Messages processed by the MQTT protocol adapter. Fifteen minute, exponentially weighted, moving average. |
| *meter.hono.mqtt.messages.processed.mean_rate*       | *host*, *tenant*, *type* | Messages processed by the MQTT protocol adapter. Mean rate of messages since the application start. |

