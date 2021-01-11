### Connect with the minIO's Pod and access files uploaded into MinIO Storage Bucket 


Step 1: Execute below command to list out MinIO's pods inside namespace "my-minio-operator"

```execute
kubectl get pods -n my-minio-operator
```

You well see output similar to this:

```


```

Step 2: Connect to the MinIO's pod  :

 Add the MinIO's podname in the below command and copy it to the terminal to execute:
 
 ```
 kubectl exec -it <podname> bash -n my-minio-operator
 ``` 

Step 3: Execute below command to check uploaded file inside test bucket which reside on containers volume mountpath: "/data"

```
cd /data/test
ls
```

Output:

```
```



