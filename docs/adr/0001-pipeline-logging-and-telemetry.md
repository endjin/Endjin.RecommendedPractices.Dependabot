# Pipeline Logging and Telemetry

## Status

Proposed

## Context

Typically CI/CD pipelines use file-based logging or integrate with the logging features of the CI/CD platform to surface progress and diagnostic information.  This presents the following challenges:

* Unstructured output
* Portability issues across CI/CD platforms
* Balancing the log verbosity in day-to-day usage, with the need for additional diagnostics in the event of an issue. This can result in having to re-run a failed pipeline in 'diagnostic' mode to get the required information, which makes it harder to troubleshoot inconsistently reproducible issues
* Limited support for metrics & telemetry
* Limited support for reporting & trend analysis over time

It is proposed that the above issues are addressed by decoupling the logging activities from the CI/CD platform - such that the CI/CD platform becomes a consumer of the underlying logging system rather than the primary publisher of the log data.

The following implementation options are considered below:

* Azure Application Insights / Azure Log Analytics
* Azure Data Lake Storage Gen2
* Azure CosmosDB (serverless) 

### Azure Application Insights / Azure Log Analytics
This options sends all log messages to an AppInsights workspace.  It would require the types of log data produced by the CI/CD workload to be mapped to the application logging semantics offered by AppInsights.

Pros:
* Built-in semantics for tracking events, exceptions etc.
* OOTB visualisation and query tools in the Azure Portal
* .Net SDK reduces up-front integration effort

Cons:
* Price premium compared to purely data storage-based pricing of other options, both for ingestion >5GB/month and retention >90 days
* Potential lag in data being available (potentially mitigated by Live Metrics Stream if supported for this scenario)
* Built-in telemetry semantics may not fully translate to CI/CD workloads (e.g. events, dependencies etc.) and their use may feel somewhat contrived
* Potentially more difficult to perform deeper or longer-term analytics


### Azure Data Lake Storage Gen2
This option sends all log message to table storage in an ADLS Gen2 storage account (to faciliate subsequent analytics). Initially the schema of the log data could be closely aligned to that of the CI/CD workloads (e.g. PowerShell streams, pipeline instances etc.) but could evolve as necessary in the future.

Pros:
* Freedom to define a telemetry scheme suitable for CI/CD workloads
* Easy to integrate with other analytical services (e.g. PowerBI, Synapse etc.)
* Lower costs for long-term retention

Cons:
* Any visualisation interface will need to be built
* Potentially more effort to integrate with CI/CD workloads


### Azure CosmosDB (serverless)
This would be similar to the ADLS Gen2 option, except using CosmosDB to take advantage of its richer query API features. 

Pros:
* Freedom to define a telemetry scheme suitable for CI/CD workloads
* Choice of APIs for integrating with CI/CD workloads
* Ability to integrate with analytical services via Synapse Link

Cons:
* Any visualisation interface will need to be built
* Potentially more effort to integrate with CI/CD workloads
* Potentially more expensive at higher throughput volumes

## Decision

TBC

## Consequences

* A CI/CD platform-agnostic approach for pipeline logging
* The ability to capture telemetry from CI/CD workloads
* Retention of telemetry beyond limits imposed by CI/CD platform
* The ability to report on pipeline activities and relationships
* The potential for applying further analytics to track/discover trends
