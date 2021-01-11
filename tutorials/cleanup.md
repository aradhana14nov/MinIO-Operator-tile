---
title: MinIO Operator cleanup Tutorial
description: This tutorial explains how to cleanup Operator
---


### Cleaning Up Operator




***Delete the operator's CRs by kubectl delete commands :***

 

Example:
 
 ```
 kubectl delete -f MinioInstance.yaml -n my-minio-operator
 ```

Note: Here MinioInstance.yaml  is the CR of the MinIO Server Instance.

Similarly,delete all the CRs.
 

***Delete the operator by kubectl delete command:***
 
 
 Example:
 
 ```
 kubectl delete -f https://operatorhub.io/install/minio-operator.yaml
 ```
 


***Deleting the CSV resource and subscription***

- Find the csv in the namespace

Example:

```
kubectl get csv -n my-minio-operator
```

Output:

```
NAME                    DISPLAY          VERSION   REPLACES   PHASE
minio-operator.v1.0.3   MinIO Operator   1.0.3                Succeeded
```

- Delete that csv :

Example:

```
kubectl delete csv/minio-operator.v1.0.3 -n my-minio-operator
```

- Get the Subscription in the same namespace 

Example:

```
kubectl get subscription -n my-minio-operator
```

Output:
```
NAME                PACKAGE          SOURCE                  CHANNEL
my-minio-operator   minio-operator   operatorhubio-catalog   stable
```


- Delete the subscription :

Example:

```
kubectl delete subscription/my-minio-operator -n my-minio-operator
```


 
***Delete all the yaml files:***
 
 Example:
 
  ```
  rm -rf MinioInstance.yaml
  ```
  

