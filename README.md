# Test storage options on Kyma Service

Testing various storage (persistence options) for SAP BTP Kyma Runtime (kubernetes service offering) 

Note: if you testing in BTP trial only a single node kyma cluster is used, so you may be exposed to issues with local node storage.


Useful links:  

[SAP BTP Kyma Runtime -- Kubernetes service](https://discovery-center.cloud.sap/serviceCatalog/kyma-runtime?region=all) 

[Kyma opensource docs](https://kyma-project.io/) 

[Kubernetes Persistent volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) 



WIP


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


## Deploy steps on Kyma
```
kubectl apply -n dev -f storage-configmap.yml
kubectl apply -n dev -f storage-local.yml
kubectl apply -n dev -f storage-aws-ebs.yml
kubectl apply -n dev -f storage-aws-ebs-csi.yml

kubectl apply -n dev -f deployment-fb.yml

```
