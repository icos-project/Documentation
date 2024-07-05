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



[1]: https://www.icos-project.eu/files/deliverables/D2.2_ICOS_Design_v1.0.pdf



