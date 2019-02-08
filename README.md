### Opereto Workers

It is recommended to create a cluster of Opereto workers in a separate k8s cluster pointing all workers to communicate with Opereto service cluster via its external Ingrees HTTPS IP address.
Following are the steps to create a workers cluster:

## Pre-configurations
1. Create a new k8s cluster (e.g opereto-workers) with kubectl command-line tool configured to communicate with that cluster.
2. Create a new user in opereto or role "Agent" for worker agents
3. Convert both the username and password to base64 (e.g. **default_user** will be converted to **ZGVmYXVsdF91c2Vy**)
4. Add opereto base64 based username and password to worker_configmap.yaml 
5. Update the following in worker.yaml:
* Update memory size (requests and limit)
* Update opereto_host - replace OPERETO_SERVICE_EXTERNAL_IP with the actual ingress URL
* Update the agent javaParams - up to half of the total pod requested memory
* Update the worker SSD storage size if needed
* Update the number of worker replicas if needed 



## grant cluster-admin permissions to the “default” service account in the kube-system namespace
```console
kubectl create clusterrolebinding add-on-cluster-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:default
```

  

## Create a user for all workers in Opereto



## Create a docker registry secret

```console
kubectl create secret docker-registry --dry-run=true regcred \
--docker-server=https://index.docker.io/v1/ \
--docker-username=<docker_hub_username> \
--docker-password=<docker_hub_password> \
--docker-email=<docker_hub_email> -o yaml > docker-secret.yaml

kubectl patch serviceaccount default -p '{"imagePullSecrets": [{"name": "regcred"}]}'

```



```console
apiVersion: v1
kind: Secret
metadata:
  name: worker-config
type: Opaque
data:
  OPERETO_PASSWORD: "add base64 encoded username here"
  OPERETO_USERNAME: "add base64 encoded password here"
```

## Create the workers

```console
kubectl create -f worker_storage.yaml
kubectl create -f worker_configmap.yaml
kubectl create -f worker_service.yaml
kubectl create -f worker.yaml
```