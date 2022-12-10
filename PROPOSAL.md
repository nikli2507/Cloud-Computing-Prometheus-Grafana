Group consisting of:
- Dario Romano - k01355351
- Daniel Lindhuber - k12013218
- Niklas Furtlehner - k12005918

For our project we choose the following topic: "Explore Prometheus & Grafana to observe your application" with the following subtasks:
  - Instrument an application to expose metrics. Set up Prometheus to collect those metrics and to alert in case of
    issues. Set up Grafana to visualize your monitored applications.
  - Demo: What can you see in Prometheus/Grafana? Run a simulation to cause an alert.
  
We structured the project into the subtasks below:
  - Installation with Helm
  - Configuration
  - Application development
  - Expose metrics in the application
  - Define Alert Rules
  - Setup of Grafana

Our application should create a textual output (e.g. random numbers) on request with vastly differing runtimes to generate every request, or changing periodically. 

Architecture diagram:
![Architecture diagram](architecture_diagram.png "Architecture diagram")
