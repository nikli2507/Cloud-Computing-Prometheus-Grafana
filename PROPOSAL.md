Group consisting of:
- Dario Romano - k01355351
- Daniel Lindhuber - k12013218
- Niklas Furtlehner - k12005918

For our project we choose the following topic: "Explore Prometheus & Grafana to observe your application" with the following subtasks:
  - Instrument an application to expose metrics. Set up Prometheus to collect those metrics and to alert in case of
    issues. Set up Grafana to visualize your monitored applications.
  - Demo: What can you see in Prometheus/Grafana? Run a simulation to cause an alert.
  
We structured the project into the subtasks below and assigned them to team members:
  - Installation with Helm - Dario, Daniel, Niklas
  - Configuration - Dario, Daniel, Niklas
  - Application development - Dario
  - Expose metrics in the application - Niklas
  - Define Alert Rules - Dario, Daniel, Niklas
  - Setup of Grafana - Daniel

Our application should create a textual output (e.g. random numbers) on request with vastly differing runtimes to generate every request, or changing periodically. 

We operate prometheus, grafana and our demo application in a kubernetes cluster. We have not yet decided on whether to use microsoft azure or google cloud but will for sure use a cloud hosted cluster and not a local one.

Prometheus is a monitoring solution that we will use to capture runtime data about our pods and services and throw alerts in our demo application. The data provided by prometheus is than visualized by grafana (which gets the data via prometheus own query interface and language). Both will be installed to our cluster via helm. Helm is a package manager for kubernetes that allows us to install all needed packages without having to manually manage dependencies (which can be a pain with prometheus and grafana). The helm package in question ("Prometheus Community Kubernetes Helm Charts") offers significant advantages in contrast to a manual installation:

// TODO

Guides: </br>
https://www.youtube.com/watch?v=QoDqxm7ybLc </br>
https://github.com/prometheus-community/helm-charts </br>
https://medium.com/globant/setup-prometheus-and-grafana-monitoring-on-kubernetes-cluster-using-helm-3484efd85891

Architecture diagram:

![architecture](https://user-images.githubusercontent.com/44208537/206863463-10790ca3-6c89-4381-911d-8479326aa295.png)
