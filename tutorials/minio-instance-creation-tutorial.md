---
title: MinIO Instance Creation tutorial
description: This tutorial explains how create Instances for your MinIO Operator.
---

### Create MinIO Instance and Sevice to access MinIO Server

Execute below command to create a secret with proper accesskey and secretkey:

```execute
kubectl create secret generic minio-creds-secret --from-literal=accesskey=admin --from-literal=secretkey=secret@123456 --namespace my-minio-operator
```


### Create below yaml file to create MinIO Operator Instance:

```execute
cat <<'EOF' > MinioInstance.yaml
apiVersion: miniocontroller.min.io/v1beta1
kind: MinIOInstance
metadata:
  name: minio
spec:
  replicas: 4
  credsSecret:
    name: minio-creds-secret
  requestAutoCert: false
EOF
```

Execute below command to create MinIO Operator instance

```execute
kubectl create -f MinioInstance.yaml -n my-minio-operator
```
You will see the following resources to be created:

```output

```

Get the associated Pods:

```execute
kubectl get pods -n my-minio-operator
```

You will be able to see the below output:

```output

```

### Create MinIO's Headless Service :

```execute
cat <<'EOF' > MinioService.yaml
apiVersion: v1
kind: Service
metadata:
  name: minio
  labels:
    app: minio
spec:
  clusterIP: None
  ports:
    - port: 9000
      name: minio
  selector:
    app: minio
EOF
```

Execute below command to create headless Service

```execute
kubectl create -f MinioService.yaml -n my-minio-operator
```

### Create NodePort Service to access MinIO's Pod 

```execute
cat <<'EOF' > MinioNodePortService.yaml
apiVersion: v1
kind: Service
metadata:
  name: minio-service
spec:
  type: NodePort
  ports:
  - port: 9000
    nodePort: 30205
  selector:
    v1beta1.min.io/instance: minio
```

Execute below command to create NodePort Service

```execute
kubectl create -f MinioNodePortService.yaml -n my-minio-operator
```


Click on below url to access pod :

Click on the <a href="http://##DNS.ip##:30205" target="_blank">http://##DNS.ip##:30205</a> to access MinIO Pod from your browser.


