## Project Vyom: Resource Recommender System

### Introduction

Project Vyom aims to optimize resource utilization in Kubernetes environments by providing intelligent CPU and memory recommendations for deployments. This document outlines the implementation details, configuration instructions, and architecture of the resource recommender system for developers, managers, and executives.

### Functional Requirements

* **Lookback Period:** The system analyzes CPU and memory usage over a configurable lookback period (default: 14 days). Recommendations are generated only after this period for a service.
* **Buffer:** An additional configurable buffer (default: 30%) is added to the recommended resource values to accommodate unpredictable workload spikes.
* **Recommendation Types:** Four recommendation types are provided based on different usage percentiles:
* **Max:** Utilizes the maximum usage observed during the lookback period.
* **p99:** Utilizes the 99th percentile usage observed during the lookback period.
* **p95:** Utilizes the 95th percentile usage observed during the lookback period.
* **p90:** Utilizes the 90th percentile usage observed during the lookback period.
* **Exclude Deployment:** Option to exclude specific deployments from the recommendation analysis.
* **Change Management:** Any resource modification triggers a new analysis period for the affected service.
* **Blue-Green Deployments:** The system ensures consistent recommendations across blue-green deployments of the same service.
* **Global Configuration:** All parameters are configurable globally at a service level, allowing fine-tuning based on specific needs.
* **Reporting:** The system generates reports in Excel format for analysis and sharing.

### Architecture

The resource recommender system comprises three main components:

1. **Data Collection:** Leverages Prometheus/Datadog to gather CPU and memory usage metrics at deployment, pod, and container levels.
2. **Recommendation Engine:** Processes the collected data, calculates resource recommendations based on configured parameters and selected recommendation type, and stores the results.
3. **API and Reporting:** Exposes the recommendations through an API and generates reports in Excel format for easy consumption.



### Implementation Details

**1. Data Collection:**

* **Recording Rules (Prometheus Example):**
* Define recording rules to calculate and store desired metrics over time.
* Example:

```
#CPU usage recording rule for deployment "my-app" in namespace "default"
- record: deployment:cpu_usage:rate5m
expr: sum(rate(container_cpu_usage_seconds_total{namespace="default", deployment="my-app"}[5m])) by (deployment)

#Memory usage recording rule for deployment "my-app" in namespace "default"
- record: deployment:memory_usage:mean5m
expr: avg(container_memory_usage_bytes{namespace="default", deployment="my-app"}) by (deployment)
```

* **Datadog (Similar Approach):**
* Utilize Datadog metrics queries and monitors to collect and aggregate CPU and memory usage data.
* Configure appropriate time windows and aggregation functions for desired metrics.

**2. Recommendation Engine:**

* The engine fetches historical usage data from Prometheus/Datadog for the defined lookback period.
* It calculates the desired percentile value (p90, p95, p99, or Max) based on the chosen recommendation type.
* The buffer percentage is added to the calculated value to determine the final recommended resource allocation.
* Logic for handling excluded deployments, change management, and blue-green deployments is implemented.

**3. API and Reporting:**

* **API:** Exposes endpoints to retrieve recommendations for specific services.
* **Reporting:** Generates Excel reports containing:
* Service name
* Current resource allocation
* Recommended resource allocation (CPU & Memory)
* Recommendation type used
* Lookback period
* Buffer percentage

### Configuration Instructions

1. **Global Configuration:**
* Define global parameters in a configuration file (e.g., YAML, JSON):
* Default lookback period
* Default buffer percentage
* Excluded deployments
* Mapping between services and their preferred recommendation types

2. **Service-Specific Overrides:**
* Allow overriding global parameters at a service level if required.

3. **Recording Rules Configuration (Prometheus):**
* Define recording rules for each relevant metric (CPU and Memory) at desired aggregation levels (deployment, pod, container).
* Store the aggregated metrics for the duration of the lookback period.

4. **Datadog Configuration:**
* Configure metrics queries, monitors, and alerts to collect and analyze CPU and Memory usage data according to the defined requirements.
* Set up appropriate time windows and aggregations for accurate analysis.

### Deployment and Usage

1. Deploy the resource recommender system alongside your Kubernetes cluster.
2. Configure Prometheus/Datadog to collect the necessary metrics and integrate with the system.
3. Set up the required global configurations and service-specific overrides.
4. Once the lookback period has elapsed for a service, recommendations will be available via the API or downloadable reports.
5. Use the recommendations to adjust resource requests and limits for deployments in your Kubernetes cluster.

### Conclusion

This document provides a comprehensive overview of the Project Vyom resource recommender system. By following the outlined implementation details and configuration instructions, you can successfully implement this system in your Kubernetes environment and optimize your resource utilization. This document should be shared with developers and managers to ensure a common understanding of the system's functionality and implementation. Enhance the content
