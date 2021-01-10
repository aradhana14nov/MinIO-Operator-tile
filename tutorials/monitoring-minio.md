---
title: Enable Monitoring to MinIO Server
description: This tutorial explains how to Enable Monitoring service to MinIO Server
---



Monitoring MinIO in Kubernetes



MinIO server exposes un-authenticated readiness and liveness endpoints so Kubernetes can natively identify unhealthy MinIO containers. MinIO also exposes Prometheus compatible data on a different endpoint to enable Prometheus users to natively monitor their MinIO deployments.

Prometheus is a cloud-native monitoring platform, built originally at SoundCloud. Prometheus offers a multi-dimensional data model with time series data identified by metric name and key/value pairs. The data collection happens via a pull model over HTTP/HTTPS. Targets to pull data from are discovered via service discovery or static configuration.

MinIO exports Prometheus compatible data by default as an authorized endpoint at /minio/prometheus/metrics. Users looking to monitor their MinIO instances can point Prometheus configuration to scrape data from this endpoint.

We will explains how to setup Prometheus and configure it to scrape data from MinIO servers by following below steps :




### Enable Monitoring service on MinIO Server

Step1: 

- Execute below command to get services of MinIO in "minio" namespace :
  
  ```execute
  kubectl get svc -n minio
  ```

 Output:
 ```
      NAME                       TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
 minio-operator-metrics   ClusterIP   10.111.158.91    <none>        8383/TCP,8686/TCP   3d4h
 minio-service            NodePort    10.106.178.202   <none>        80:30685/TCP        3d4h
 ```
 
- Get the port of minio-service :
  
  From above command output, minio-service port is 30685 

- To enable monitoring using Prometheus exporter pod and service, create the below yaml definition of the Custom Resource:


```execute
cat <<'EOF'> MinIOmonitoring.yaml
apiVersion: minio.persistentsys/v1alpha1
kind: Monitor
metadata:
  name: minio-monitor
spec:
  # Add fields here
  size: 1
  # Database source to connect with for colleting metrics
  # Format: "<db-user>:<db-password>@(<dbhost>:<dbport>)/<dbname>">
  # Make approprite changes 
  dataSourceName: "root:password@(##DNS.ip##:30685)/test-db"
  # Image name with version
  # Refer https://registry.hub.docker.com/r/prom/mysqld-exporter for more details
  image: "prom/mysqld-exporter"
EOF
```

Note: The database host and port should be correct for metrics to work.



Step2: Execute below command to Create Instance of Monitoring: 

```execute
kubectl create -f MinIOmonitoring.yaml -n minio
```

Output:


```
monitor.minio.persistentsys/minio-monitor created
```

This will start Prometheus exporter pod and service. 




## Install Prometheus Operator and Deploy Prometheus Instance and ServiceMonitor 


If you already have Prometheus Operator installed, you can skip installing Prometheus and  proceed with next step.


### Install Prometheus operator and Deploy Prometheus Instance and ServiceMonitor :
 

Step 1 : Install the Prometheus operator by running the following command:

```execute
kubectl create -f https://operatorhub.io/install/prometheus.yaml
```

Output:

```
subscription.operators.coreos.com/my-prometheus created
```

This Operator will be installed in the "operators" namespace and will be usable from all namespaces in the cluster.


- After Operator installation, verify that your operator got successfully installed by executing the below command:

```execute
kubectl get csv -n operators
```

You should see a similar output as below:

```
NAME                        DISPLAY               VERSION   REPLACES                    PHASE
prometheusoperator.0.37.0   Prometheus Operator   0.37.0    prometheusoperator.0.32.0   Succeeded
```

From above output, once operator is successfully installed, **PHASE** will be as "Succeeded" 

- Check the Pods status using below command:

```execute
kubectl get pods -n operators
```

OutputYou should see a similar output as below:

```
NAME                                   READY   STATUS    RESTARTS   AGE
prometheus-operator-6f7589ff7f-wq9zd   1/1     Running   0          8m35s
```

In above output, STATUS as "Running" shows the pods are up and running.


If you are installing from operatorhub, then by default it installs the operator in operators namespace.
Below steps assumes that its deployed in operators namespace.



Step 2: Create below yaml definition of the Custom Resource to create a Prometheus Instance along with a ServiceAccount and a Service of Type NodePort :


```execute
cat <<'EOF' > prometheusInstance.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: server
  labels:
    prometheus: k8s
  namespace: operators
spec:
  replicas: 1
  serviceAccountName: prometheus-k8s
  securityContext: {}
  serviceMonitorSelector:
    matchLabels:
      app: playground  
  alerting:
    alertmanagers:
      - namespace: operators
        name: alertmanager-main
        port: web  
EOF
```


Step 3: Execute below command to create Prometheus instance:



```execute
kubectl create -f prometheusInstance.yaml -n operators
```

Output:

```
prometheus.monitoring.coreos.com/server created
```


- Check the pods status using below commands:



```execute
kubectl get pods -n operators
```

You should see a similar output as below:

```
NAME                                   READY   STATUS    RESTARTS   AGE
prometheus-operator-6f7589ff7f-wq9zd   1/1     Running   0          14m
prometheus-server-0                    3/3     Running   1          40s
```

Step 4: Create below yaml definition of the Custom Resource to create the service to access prometheus server:


```execute
cat <<'EOF' > prometheus_service.yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
spec:
  type: NodePort
  ports:
  - name: web
    nodePort: 30100
    port: 9090
    protocol: TCP
    targetPort: web
  selector:
    app: prometheus
EOF
```

- Execute below command to create Prometheus Service:

```execute
kubectl create -f prometheus_service.yaml -n operators
```

Output:

```
service/prometheus created
```

- Access the service :


Access the Prometheus dashboard using below link:

http://##DNS.ip##:30100



Step 5: Create below yaml definition of the Custom Resource to create Instance of ServiceMonitor:


```execute
cat <<'EOF' > ServiceMonitor.yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: minio-monitor 
  labels:
    app: playground
  namespace: operators
spec:
  namespaceSelector:
    any: true
  selector:
    matchLabels:
      tier: monitor-app 
  endpoints:
   - interval: 20s
     port: monitor    
EOF
```


Step 6: Execute below command to create ServiceMonitor instance :


```execute
kubectl create -f ServiceMonitor.yaml -n operators
```

Output:

```
servicemonitor.monitoring.coreos.com/minio-monitor created
```


- Check the pods status using below commands:



```execute
kubectl get pods -n operators
```



### Step 2: Verify prometheus monitoring metrics :

Access the Prometheus dashboard using below link:

http://##DNS.ip##:30100

- On the prometheus UI, Go to Status -> Targets to see endpoints.


 ![](_images/targets.PNG)



- From the dropdown you can select the query and click on "Execute" to see MinIO Metrics. See below snapshot :


![](_images/queryexecution.PNG)




