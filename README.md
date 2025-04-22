# DatDog-fundamentals

## Agent Architecture
The main components to this process are:

- The **Collector**: which runs checks and collects metrics.
- The **Forwarder**: which sends payloads to Datadog.

* Two optional processes are spawned by the Agent if enabled in the datadog.yaml configuration file:

- The `APM` Agent is a process that collects traces. It is enabled by default.
- The `Process Agent` is a process that collects live process information. By default, the Process Agent only collects available containers, otherwise it is disabled.


---

By default the Agent binds three ports on Linux and four ports on Windows and macOS:

|Port	| Description|
-------------------------
| 5000	| Exposes runtime metrics about the Agent.| 
| 5001	| Used by the Agent CLI and GUI to send commands and pull information | from the running Agent.
| 5002	| Serves the GUI server on Windows and macOS. or security reasons, the GUI can only be accessed from the local network interface (localhost/127.0.0.1)| 
| 8125	| Used for the DogStatsD server to receive external metrics.| 
---

### Collector

The collector gathers all standard metrics every 15 seconds. Agent 6 embeds a Python 2.7 interpreter to run integrations and [custom checks](https://docs.datadoghq.com/developers/custom_checks/write_agent_check/).

##### custom checks 
To build a custom checks use a yaml and py file in check file in the `checks.d`
**Note**:
- The names of the configuration and check files must match
    - If your check is called `custom_checkvalue.py`, your configuration file 
      must be named `custom_checkvalue.yaml`.
- The check file must be readable and executable by the Agent user

### Forwarder

The Agent forwarder sends metrics over HTTPS to Datadog. Buffering prevents network splits from affecting metrics reporting. Metrics are buffered in memory until a limit in size or number of outstanding send requests is reached. Afterward, the oldest metrics are discarded to keep the forwarder’s memory footprint manageable. Logs are sent to Datadog over an SSL-encrypted TCP connection.

### DogStatsD

In Agent 6, DogStatsD is a Golang implementation of Etsy’s StatsD metric aggregation daemon. DogStatsD receives and rolls up arbitrary metrics over UDP or a UNIX socket, allowing custom code to be instrumented without adding latency. Learn more about [DogStatsD](https://docs.datadoghq.com/metrics/custom_metrics/dogstatsd_metrics_submission/). 

- While StatsD accepts only metrics, DogStatsD accepts all three of the major Datadog data types: metrics, events, and service checks.


Agent Commands

[Agent Commands](https://docs.datadoghq.com/agent/configuration/agent-commands/)

