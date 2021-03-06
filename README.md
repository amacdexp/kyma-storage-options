# Test storage options on Kyma Service

Testing various storage (persistence options) for Kyma and SAP BTP Kyma Runtime (kubernetes service offering)  using 
[filebrowser](https://filebrowser.org/installation) 

Note: If testing in BTP trial only a single node kyma cluster is used, so you may be exposed to other issues with local node storage. 

Note2: Currently creation of PV and Storage Classes is restricted on BTP Kyma Runtime.


## filebrowser running on Kyma Service
![alt text](https://github.com/amacdexp/kyma-storage-options/blob/main/srv/filebrowser_on_Kyma.png?raw=true)


## filebrowser running on SAP BTP Kyma Runtime Service
![alt text](https://github.com/amacdexp/kyma-storage-options/blob/main/srv/filebrowser_on_BTP_Kyma_Runtime.png?raw=true)




## Useful links:  

[SAP BTP Kyma Runtime -- Kubernetes service](https://discovery-center.cloud.sap/serviceCatalog/kyma-runtime?region=all) 

[Kyma opensource docs](https://kyma-project.io/) 

[Kubernetes Persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) 

This also utilises the following image to provide S3 access via an NFS server: 

[amacdexp/nfs-server-s3-support](https://github.com/amacdexp/nfs-server-s3-support) 
[Dockerhub amacdexp/nfs-server-s3-support](https://hub.docker.com/repository/docker/amacdexp/nfs-server-s3-support)




## local test of filebrowser 
(https://filebrowser.org/installation) 

```

export FB_PATH="<LOCATION OF gitclone>/kyma-storage-options"
mkdir $FB_PATH/srv

echo ''' 
{
  "port": 80,
  "baseURL": "",
  "address": "",
  "log": "stdout",
  "database": "/database.db",
  "root": "/srv"
}
''' > .filebrowser.json


touch filebrowser.db


docker run \
    -v $FB_PATH/srv:/srv \
    -v $FB_PATH/filebrowser.db:/database.db \
    -v $FB_PATH/.filebrowser.json:/.filebrowser.json \
    --user $(id -u):$(id -g) \
    -p 80:80 \
    filebrowser/filebrowser


```


## Deploy filebrowser on Kyma, demonstrating various storage options
```
#Basic storage
kubectl apply -n dev -f storage-configmap.yml

#update storage-local.yaml with a single node from the cluster: 'kubectl get nodes'
kubectl apply -n dev -f storage-local.yml
kubectl apply -n dev -f storage-default-pvc.yml
kubectl apply -n dev -f storage-aws-ebs.yml
kubectl apply -n dev -f storage-aws-ebs-csi.yml

#NFS (2 Servers on Kubernetes: 1 Normal with PVC , 1 with S3 mount)
kubectl apply -n dev -f nfs-server.yml

kubectl -n dev create secret generic aws-s3 \
  --from-literal=AWS_KEY='your AWS key' \
  --from-literal=AWS_SECRET_KEY='your AWS secret'
## update the nfs-server-s3.yml  with your AWS region / bucket
kubectl apply -n dev -f nfs-s3-server.yml

## update the cluster ip address of the servers in the following yaml
kubectl apply -n dev -f storage-nfs.yml
kubectl apply -n dev -f storage-nfs-s3.yml



#busybox for testing pvc's seperately
kubectl apply -n dev -f pod-busybox.yml
kubectl -n dev describe pod busybox
kubectl exec -n dev --stdin --tty busybox -- /bin/sh
kubectl delete -n dev pod busybox --force


#filebrowser with pvc's mounted 
#Update APIRule host  with appropriate <cluster> id 
#OPT: update local volume mount path with node suffix
kubectl apply -n dev -f deployment-fb.yml

kubectl -n dev describe pod $(kubectl -n dev get pods -l app=filebrowser --field-selector=status.phase=Pending -o jsonpath="{.items[0].metadata.name}")


```

## Deploy filebrowser on SAP BTP Kyma Runtime, demonstrating current storage options
```
#Basic storage
kubectl apply -n dev -f storage-configmap.yml
kubectl apply -n dev -f storage-default-pvc.yml





#NFS (2 Servers on Kubernetes: 1 Normal with PVC , 1 with S3 mount)
kubectl apply -n dev -f nfs-server.yml

kubectl -n dev create secret generic aws-s3 \
  --from-literal=AWS_KEY='your AWS key' \
  --from-literal=AWS_SECRET_KEY='your AWS secret'
## update the nfs-server-s3.yml  with your AWS region / bucket
kubectl apply -n dev -f nfs-s3-server.yml


#filebrowser with pvc's mounted 
#Update APIRule host  with appropriate <cluster> id 
#OPT: update local volume mount path with node suffix
kubectl apply -n dev -f deployment-fb-btp.yml

kubectl -n dev describe pod $(kubectl -n dev get pods -l app=filebrowser --field-selector=status.phase=Pending -o jsonpath="{.items[0].metadata.name}")

```
