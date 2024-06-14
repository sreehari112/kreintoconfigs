## Project Vyom: Kubernetes Resource Recommender System

This document details Project Vyom, an intelligent resource recommender system designed to optimize CPU and memory utilization in Kubernetes environments. It provides a clear understanding of the system's functionality, architecture, and implementation for developers, managers, and executives.

### Executive Summary

Project Vyom addresses the challenge of manually tuning resource requests and limits in Kubernetes by providing data-driven CPU and memory recommendations. This leads to optimized resource utilization, cost savings, and improved application performance.

### Functional Requirements

* **Historical Analysis:** Analyzes resource usage over a configurable lookback period (default: 14 days) to ensure recommendations are based on actual workload patterns.
* **Adaptive Buffer:** Applies a configurable buffer (default: 30%) to the recommended resource values to handle unforeseen workload spikes, ensuring application stability.
* **Flexible Recommendation Strategies:** Offers multiple recommendation types based on usage percentiles (Max, p99, p95, p90) to cater to different risk tolerances and performance requirements.
* **Granular Control:** Allows excluding specific deployments from analysis and provides service-level overrides for global parameters, enabling fine-grained control and customization.
* **Seamless Integration:** Integrates with existing monitoring solutions like Prometheus or Datadog for data collection, minimizing implementation overhead.
* **Deployment Agnostic:** Handles blue-green deployments seamlessly, ensuring consistent recommendations across different deployment instances of the same service.
* **Change-Aware Analysis:** Automatically triggers a new analysis period upon resource modification, ensuring recommendations remain relevant and up-to-date.
* **Actionable Insights:** Generates comprehensive reports in Excel format, facilitating data analysis, communication, and decision-making processes.

### Technical Architecture

Project Vyom comprises three interconnected components:

1. **Data Ingestion Layer:** Leverages existing monitoring tools like Prometheus or Datadog to collect CPU and memory usage metrics at various levels (deployment, pod, container) based on configurable recording rules or metrics queries.
2. **Recommendation Engine:** Processes the collected historical data, applies the chosen recommendation strategy (percentile-based or Max), adds the configured buffer, and generates optimized resource recommendations for each analyzed service. This component also incorporates logic for excluded deployments, change management, and blue-green deployments.
3. **Presentation & Access Layer:** Provides access to the generated recommendations through a user-friendly API for seamless integration with other tools and workflows. Additionally, this layer generates comprehensive reports in Excel format for easy analysis and sharing.

![Architecture Diagram](https://i.imgur.com/nR87b7O.png)

### Implementation Details

**1. Data Ingestion:**

* **Prometheus Integration:** Leverages recording rules to calculate and store desired metrics (e.g., average CPU usage, maximum memory consumption) over the configured lookback period.
* **Datadog Integration:** Utilizes Datadog metrics queries and monitors to collect, aggregate, and store similar CPU and memory usage data based on pre-defined time windows and aggregation functions.

**2. Recommendation Engine:**

* Retrieves historical usage data from the chosen monitoring system (Prometheus or Datadog) for the defined lookback period and target service.
* Calculates the desired percentile value (p90, p95, p99, or Max) based on the configured recommendation type for the service.
* Applies the defined buffer percentage to the calculated percentile value, generating the final recommended resource allocation for CPU and memory.
* Implements logic to exclude specific deployments from analysis, manage changes in resource allocation, and ensure consistent recommendations across blue-green deployments.

**3. Presentation & Access:**

* **API:** Exposes REST endpoints to retrieve resource recommendations for specific services on demand, enabling integration with automation scripts, CI/CD pipelines, and other tools.
* **Reporting:** Generates detailed reports in Excel format containing:
* Service name
* Current resource allocation
* Recommended resource allocation (CPU & Memory)
* Chosen recommendation type
* Lookback period used
* Applied buffer percentage

### Deployment and Configuration

1. **Deployment:** Deploy Project Vyom alongside your existing Kubernetes cluster using standard deployment methods (e.g., Helm charts, YAML manifests).
2. **Monitoring Integration:** Configure Prometheus or Datadog to collect the necessary CPU and memory metrics according to the predefined recording rules or metrics queries and ensure data is accessible by Project Vyom.
3. **Global Configuration:** Define global parameters in a centralized configuration file (e.g., YAML, JSON), including:
* Default lookback period
* Default buffer percentage
* List of globally excluded deployments
* Default recommendation type for all services
4. **Service-Specific Overrides:** Configure service-level overrides for any global parameter directly within Project Vyom's configuration interface, providing fine-grained control over individual services.
5. **Recommendation Consumption:** Retrieve resource recommendations through the provided API or access the generated reports to analyze and apply the optimized resource allocations to your Kubernetes deployments.

### Benefits and Impact

* **Optimized Resource Utilization:** Right-sizes resource requests and limits based on actual workload patterns, leading to efficient utilization of cluster resources.
* **Cost Savings:** Reduces unnecessary resource allocation and cloud spending by preventing over-provisioning and ensuring applications only consume what they need.
* **Improved Application Performance:** Minimizes resource contention and ensures applications have sufficient resources to function optimally, leading to better performance and user experience.
* **Reduced Management Overhead:** Automates resource recommendations, freeing up engineers from manual tuning and allowing them to focus on other critical tasks.
* **Data-Driven Decision Making:** Provides actionable insights through detailed reports, enabling informed decision-making regarding resource allocation and capacity planning.

### Conclusion

Project Vyom empowers organizations to implement efficient resource management practices in their Kubernetes environments. By leveraging historical data, configurable parameters, and a robust architecture, Project Vyom delivers optimized resource recommendations, leading to cost savings, improved performance, and reduced management overhead. This document serves as a comprehensive guide for implementing and leveraging the power of Project Vyom within your organization.