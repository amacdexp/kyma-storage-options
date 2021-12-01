# Test storage options on Kyma Service

Testing various storage (persistence options) for SAP BTP Kyma Runtime (kubernetes service offering) 

Note: if you testing in BTP trial only a single node kyma cluster is used, so you may be exposed to issues with local node storage.
Note: Currently creation of PV and Storage Classes is restricted on BTP Kyma Service.


Useful links:  

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


## Deploy filebrowser on Kyma demonstrating various storage options
```
#Basic storage
kubectl apply -n dev -f storage-configmap.yml

#update storage-local.yaml with a single node from the cluster: 'kubectl get nodes'
kubectl apply -n dev -f storage-local.yml
kubectl apply -n dev -f storage-default-pvc.yml
kubectl apply -n dev -f storage-aws-ebs.yml
kubectl apply -n dev -f storage-aws-ebs-csi.yml

#NFS (on Kubernetes with S3 support)
kubectl apply -n dev -f nfs-server.yml
kubectl apply -n dev -f nfs-s3-server.yml

## update cluster ip address of the servers in the following yaml
kubectl apply -n dev -f storage-nfs.yml
kubectl apply -n dev -f storage-nfs-s3.yml



#busybox for testing pvc's seperately
kubectl apply -n dev -f pod-busybox.yml
kubectl -n dev describe pod busybox
kubectl exec -n dev --stdin --tty busybox -- /bin/sh
kubectl delete -n dev pod busybox --force


#filebrowser with pvc's mounted
kubectl apply -n dev -f deployment-fb.yml

kubectl -n dev describe pod $(kubectl -n dev get pods -l app=filebrowser --field-selector=status.phase=Pending -o jsonpath="{.items[0].metadata.name}")


```
