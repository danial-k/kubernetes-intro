# Kubernetes Introduction
An introduction to Kubernetes, assuming you are running a local Kubernetes cluster.
## Dashboard
Apply the dashboard:
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```
 Start proxy:
 ```shell
 kubectl proxy
 ```
 
 Explore the API (download ```swagger.json``` from http://127.0.0.1/swagger.json):
 ```shell
docker run \
--name redoc-api-k8s \
-p 9001:80 \
--mount type=bind,src=`pwd`/swagger.json,dst=/usr/share/nginx/html/swagger.json \
-e SPEC_URL='swagger.json' \
redocly/redoc
 ```
 
 Extract bearer token for web login:
 ```shell
 kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kubernetes-dashboard-token | awk '{print $1}')
 ```
 
 Log in with bearer token at http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/.
 
 ## Creating a deployment and service
 In the top-right of the dashboard, click create and enter the following yaml:
 ```yaml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: whoami-deployment
spec:
  selector:
    matchLabels:
      app: whoami
  replicas: 3 # tells deployment to run 3 pods matching the template
  template:
    metadata:
      labels:
        app: whoami
    spec:
      containers:
      - name: whoami
        image: jwilder/whoami:latest
        ports:
        - containerPort: 8000
 ```
 
This will create a replication set with 3 pods maintained.  To expose the pods as a load balanced externally available service, create the following: 
 ```yaml
apiVersion: v1
kind: Service
metadata:
  name: whoami-service
spec:
  selector:
    app: whoami
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 8002
    targetPort: 8000
 ```

Verify load balancing by running multiple instance of:
```shell
curl -s "http://127.0.0.1:8002/?[1-10]"
```
