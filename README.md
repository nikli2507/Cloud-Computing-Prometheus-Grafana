# Cloud-Computing-Prometheus-Grafana

## Installation

Create a kubernetes cluster in a cloud environment or locally (e.g. with minikube) and connect it with kubectl.
For our project we chose google cloud.

### Helm
Before we can start in the cluster itself we first have to install the helm package manager.
See https://helm.sh/docs/intro/install/.

### Helm Repositories
Repository for the Prometheus-Stack.
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```
Repository for the MQTT-Exporter.
```
helm repo add k8s-at-home https://k8s-at-home.com/charts/
```
After that update your repositories.
```
helm repo update
```

### Prometheus-Stack
Install the stack with the following command:
```
helm install prometheus prometheus-community/kube-prometheus-stack
```
This installs the prometheus operator, the grafana dashboard and all configuration files and scripts that are needed for the prometheus monitoring to function properly on a kubernetes cluster.
</br>
The installation can be checked with
```
kubectl get all
```
Your output should look similar to this:
```
NAME                                                         READY   STATUS    RESTARTS        AGE
pod/alertmanager-prometheus-kube-prometheus-alertmanager-0   2/2     Running   1 (3d2h ago)    4d2
pod/prometheus-grafana-54b79445c7-kndjq                      3/3     Running   0               3d
pod/prometheus-kube-prometheus-operator-55cdd486b8-zmnlr     1/1     Running   0               4d20h
pod/prometheus-kube-state-metrics-675b965b4c-h896g           1/1     Running   0               4d20h
pod/prometheus-prometheus-kube-prometheus-prometheus-0       2/2     Running   0               4d20h
pod/prometheus-prometheus-node-exporter-nl6vm                1/1     Running   0               3d2h
pod/prometheus-prometheus-node-exporter-t6n7k                1/1     Running   0               3d2h
pod/prometheus-prometheus-node-exporter-txnrx                1/1     Running   0               3d2h

NAME                                              TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
service/alertmanager-operated                     ClusterIP      None          <none>         9093/TCP,9094/TCP,9094/UDP   4d22h
service/kubernetes                                ClusterIP      10.0.0.1      <none>         443/TCP                      4d22h
service/prometheus-grafana                        ClusterIP      10.0.3.74     <none>         80/TCP                       4d22h
service/prometheus-kube-prometheus-alertmanager   ClusterIP      10.0.5.81     <none>         9093/TCP                     4d22h
service/prometheus-kube-prometheus-operator       ClusterIP      10.0.2.238    <none>         443/TCP                      4d22h
service/prometheus-kube-prometheus-prometheus     ClusterIP      10.0.9.71     <none>         9090/TCP                     4d22h
service/prometheus-kube-state-metrics             ClusterIP      10.0.15.78    <none>         8080/TCP                     4d22h
service/prometheus-operated                       ClusterIP      None          <none>         9090/TCP                     4d22h
service/prometheus-prometheus-node-exporter       ClusterIP      10.0.8.2      <none>         9100/TCP                     4d22h

NAME                                                 DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/prometheus-prometheus-node-exporter   3         3         3       3            3           <none>          4d22h

NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-grafana                    1/1     1            1           4d22h
deployment.apps/prometheus-kube-prometheus-operator   1/1     1            1           4d22h
deployment.apps/prometheus-kube-state-metrics         1/1     1            1           4d22h

NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-grafana-54b79445c7                    1         1         1       4d22h
replicaset.apps/prometheus-kube-prometheus-operator-55cdd486b8   1         1         1       4d22h
replicaset.apps/prometheus-kube-state-metrics-675b965b4c         1         1         1       4d22h

NAME                                                                    READY   AGE
statefulset.apps/alertmanager-prometheus-kube-prometheus-alertmanager   1/1     4d22h
statefulset.apps/prometheus-prometheus-kube-prometheus-prometheus       1/1     4d22h
```

### MQTT
#### Server
deployment_server.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: server
  template:
    metadata:
      labels:
        app: server
    spec:
      containers:
      - name: server
        image: eclipse-mosquitto:1.6.15
        ports:
        - containerPort: 1883
```
Install the server on your cluster with the command:
```
kubectl apply -f deployment_server.yaml
```

#### Clients
deployment_client1.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: client1
  template:
    metadata:
      labels:
        app: client1
    spec:
      containers:
      - name: client1
        image: efrecon/mqtt-client
        command: ["mosquitto_pub"]
        args: ["-p", "1883", "-h", "10.0.8.44", "-t", "goebe_bam_gurken", "-m", "{\"feinste_tiroler_bananen_num\":\"10\"}", "--repeat", "2000", "--repeat-delay", "10"]
        ports:
        - containerPort: 42213
```

Create a second client with different runtime and value("deployment_client2.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: client2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: client2
  template:
    metadata:
      labels:
        app: client2
    spec:
      containers:
      - name: client2
        image: efrecon/mqtt-client
        command: ["mosquitto_pub"]
        args: ["-p", "1883", "-h", "10.0.8.44", "-t", "goebe_bam_gurken", "-m", "{\"feinste_tiroler_bananen_num\":\"12\"}", "--repeat", "2000", "--repeat-delay", "30"]
        ports:
        - containerPort: 42214
```

Install both clients on your cluster with the commands:
```
kubectl apply -f deployment_client1.yaml
kubectl apply -f deployment_client2.yaml
```

### Expose Server with Services
To have a stable server ip to which the clients can send their data we need to make a service of type "clusterIP":
service_server.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: server
  labels:
    app: server
spec:
  ports:
  - port: 1883
    protocol: TCP
    targetPort: 1883
  selector:
    app: server
  type: ClusterIP
```
This ip is only internal and because we want to use MQTT clients outside of the cluster we need to expose it to the outside.
We do this by creating a service of type "LoadBalancer":
service_server_outside.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: server-outside
  labels:
    app: server
spec:
  ports:
  - port: 1884
    protocol: TCP
    targetPort: 1883
  selector:
    app: server
  type: LoadBalancer
```
Deploy both services with the commands:
```
kubectl apply -f service_server.yaml
kubectl apply -f service_server_outside.yaml
```

By calling
```
kubectl describe service server
```
One can look up the internal ip that we already used in the clients deployment files to address our server.

Calling 
```
kubectl describe service server-outside
```
shows you the ingress (the outside) ip of the LoadBalancer.
With this ip and port you can address the server from outside the cluster.
```yaml
Name:                     server-outside
Namespace:                default
Labels:                   app=server
Annotations:              cloud.google.com/neg: {"ingress":true}
Selector:                 app=server
Type:                     LoadBalancer
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.0.1.58
IPs:                      10.0.1.58
LoadBalancer Ingress:     34.140.215.4
Port:                     <unset>  1884/TCP
TargetPort:               1883/TCP
NodePort:                 <unset>  32284/TCP
Endpoints:                10.124.0.16:1883
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Exporter
To install the exporter we first have to create config file.
We can get the default config file by using the command:
```
helm show values k8s-at-home/mqtt-exporter > values.yaml
```

We can now edit the file to look like this (all important points are commented):
```yaml
#
# IMPORTANT NOTE
#
# This chart inherits from our common library chart. You can check the default values/options here:
# https://github.com/k8s-at-home/library-charts/tree/main/charts/stable/common/values.yaml
#

image:
  # -- image repository
  repository: kpetrem/mqtt-exporter
  # -- image tag
  tag: latest
  # -- image pull policy
  pullPolicy: IfNotPresent

# -- environment variables. See [image docs](https://developer.us-1.veritone.com/machinebox/overview) for more details.
# @default -- See below
env:
  # -- Set the container timezone
  TZ: UTC
  # -- Logging level
  LOG_LEVEL: DEBUG
  # -- Comma-separated lists of topics to ignore. Accepts wildcards.
  MQTT_IGNORED_TOPICS:
  # -- IP or hostname of MQTT broker
  MQTT_ADDRESS: 10.0.8.44
  # -- TCP port of MQTT broker
  MQTT_PORT: 1883
  # -- Topic path to subscribe to
  MQTT_TOPIC: "#"
  # -- Keep alive interval to maintain connection with MQTT broker
  MQTT_KEEPALIVE: 60
  # -- Username which should be used to authenticate against the MQTT broker
  MQTT_USERNAME:
  # -- Password which should be used to authenticate against the MQTT broker
  MQTT_PASSWORD:
  # -- HTTP server PORT to expose Prometheus metrics
  PROMETHEUS_PORT: 9000
  # -- Prefix added to the metric name, example: mqtt_temperature
  PROMETHEUS_PREFIX: mqtt_
  # --  Define the Prometheus label for the topic, example temperature{topic="device1"}
  TOPIC_LABEL: topic
  # --  Normalize sensor name for device availability metric added by Zigbee2MQTT
  ZIGBEE2MQTT_AVAILABILITY: "False"
  LOG_MQTT_MESSAGE: "True"

# -- Configures service settings for the chart.
# @default -- See values.yaml
service:
  main:
    ports:
      http:
        port: 9000

metrics:
  # -- Enable and configure a Prometheus serviceMonitor for the chart under this key.
  # @default -- See values.yaml
  enabled: true
  serviceMonitor:
    # -- Interval at which Prometheus should scrape metrics
    interval: 1s
    # -- Timeout after which the scrape is ended
    scrapeTimeout: 500ms
    # -- Additional labels for the Kubernetes `ServiceMonitor` object
    labels:
      release: prometheus

ingress:
  # -- Enable and configure ingress settings for the chart under this key.
  # @default -- See values.yaml
  main:
    enabled: false

# -- Configure persistence settings for the chart under this key.
# @default -- See values.yaml
persistence:
  config:
    enabled: false
    mountPath: /conf

securityContext:
  capabilities:
    drop:
      - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 65534
```
With this settings the installation process also create a ServiceMonitor and the MQTT credentials are saved.
</br>
To finally install the exporter we execute the following command:
```
helm install mqtt-exporter k8s-at-home/mqtt-exporter -f values.yaml
```

### Grafana
Now that the server is up and running, the clients send data and our exporter pulls the metrics from the server and exposes them to prometheus, we can finally look into grafana.
To acess grafana, we have to expose it with a port-forward:
```
kubectl port-forward deployment/prometheus-grafana 3000
```
The default credentials for the login can be found in the documentation: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack
</br>
They currently are:
- username: admin
- password: prom-operator
</br>
Now grafana dashboards can be created and alerts configured with the (self explaining) alert UI.
</br>
The only thing to know is that our metrics are (like configured in values.yaml) prefixed with "mqtt_".



