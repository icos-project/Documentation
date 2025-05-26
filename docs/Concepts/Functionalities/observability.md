# Observability
Monitoring a large and diverse set of distributed computing resources and software generates a large
amount of data of various types and may require data to be analysed, pre-processed, and aggregated if
necessary, and transmitted from edge nodes / IoT devices where monitored services are deployed to
locations where end-to-end services and resources management decisions are made.
To prevent resource and application monitoring data from further contributing to the data overload,
intelligent mechanisms are needed to automatically and dynamically decide what data to collect (i.e., 
what type of monitoring data to measure), the granularity of the data (i.e., what level of information to
collect for a given item), and the frequency of the data (i.e., the interval between two collections for a
given item). 

The **Logging and Telemetry** is responsible for collecting and uploading telemetry reports into ICOS Shell. 
Such logs shall be accessible both locally from within the edge device and remotely from the ICOS Shell. 
This component will also support the exchange of metrics and information between the Meta-kernel and user 
workloads ( [*D2.2 ICOS Architecture and Design (IT-1)*][1] ).

The figure highlights the main third-party technologies integrated to realize the telemetry components.
With respect to the previous version presented in D3.1 [1], the following new third party technologies
have been used:

- The **Telemetry Gateway** component has been implemented using the OpenTelemetry and Prometheus
protocols and tools. This ensures a perfect compatibility and uniformity with the other telemetry
components.

- The **Alerting API** has been implemented using two components:

	1. [Prometheus AlertManager](https://prometheus.io/docs/alerting/latest/alertmanager/) (Apache 2.0 license) to send alerts.
		

	2. [Prometheus-API](https://github.com/hayk96/prometheus-api) project (MIT license) to manage alerting rules programmatically.
		
<figure markdown="1">
<img src="../../../assets/images/logtelemetry.png" style="align:center">
</figure>

In order to avoid confusion and clashes with ICOS concepts and terminology (e.g., the word Agent is
use in both contexts), a renaming of the software module has been done. In particular:

- **Telemetruum Hub** is the software module that implements the Telemetry Controller architectural
component.

- **Telemetruum Gateway** is the software module that implements the Telemetry Gateway architectural
component.

- **Telemetruum Leaf** is the software module that implements the Telemetry Agent architectural
component.



[1]: https://www.icos-project.eu/files/deliverables/D2.2_ICOS_Design_v1.0.pdf



