# Continuum Management

The Continuum Management is responsible for bootstrapping new devices into the ICOS continuum. 
It provides the mechanisms for: 

+ registering and activating the edge devices into ICOS Shell; 
+ enabling the remote execution of management jobs, issued by users, allowing for on-demand management and 
  re-configuration of the Meta-Kernel itself, as well as low-level operations such as real time configuration, 
  kernel tunning or updates/upgrades (including mechanisms to automatically enforce green strategies such as: 
  offloading of computation-intensive tasks from end users to edge nodes or the cloud,configuring edge-device 
  CPU processor, connectivity and bandwidth utilization, resources location, etc.); 
+ configuring the edge devices network such that secure remote access, multi-access edge computing and network
  attack surface protection can be enabled; 
+ ensuring that the user workloads and the underlying device are performing according to the expectations 
  mandated by pre-defined and configurable SLA criteria.
  The component also makes sure that failed workloads are rebalanced and quality is reinforced, and; 
+ discovering, labeling and categorizing both the underlying device and all relevant peripherals such as 
  IoT sensors and actuators.