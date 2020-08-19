# KEDA for custom metrics based HPA


[KEDA](https://keda.sh/) is a Kubernetes-based Event Driven Autoscaler. With `KEDA`, you can drive the scaling of any container in Kubernetes based on the number of events needing to be processed.

The following demo is based on the examples [here](https://github.com/abhirockzz/kubernetes-keda-prometheus)

## Install KEDA
Follow setups in [KEDA docs](https://keda.sh/docs/1.5/deploy/#install)

I used Helm 3 method
```
kubectl create namespace keda
helm install keda kedacore/keda --namespace keda
```

## Install Prometheus

```
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - services
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: default
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: default
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: prom-conf
  labels:
    name: prom-conf
data:
  prometheus.yml: |-
    global:
      scrape_interval: 5s
      evaluation_interval: 5s
    scrape_configs:
      - job_name: 'go-prom-job'

        kubernetes_sd_configs:
        - role: service
        relabel_configs:
        - source_labels: [__meta_kubernetes_service_label_run]
          regex: go-prom-app-service
          action: keep
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: prometheus-deployment
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prom-conf
  
        - name: prometheus-storage-volume
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
spec:
  ports:
  - port: 9090
    protocol: TCP
  selector:
    app: prometheus-server
```

### Check prometheus from UI
+ `kubectl port-forward service/prometheus-service 9090`
+ open `localhost:9090` in a browser

## Install redis for the demo app
`helm install redis-server --set cluster.enabled=false --set usePassword=false stable/redis`

## Deploy demo app

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-prom-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-prom-app
  template:
    metadata:
      labels:
        app: go-prom-app
    spec:
      containers:
      - name: go-prom-app
        image: abhirockzz/go-prom-app
        env:
          - name: REDIS_HOST
            value: redis-server-master.default.svc.cluster.local
          - name: REDIS_PORT
            value: "6379"
        imagePullPolicy: Always
        ports:
            - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: go-prom-app-service
  labels:
    run: go-prom-app-service
spec:
  ports:
  - port: 8080
    protocol: TCP
  selector:
    app: go-prom-app
```
Make sure you can see the demo app in the prometheus targets page `http://localhost:9090/targets`

## Deploy KEDA ScaledObject resource

```
apiVersion: keda.k8s.io/v1alpha1
kind: ScaledObject
metadata:
  name: prometheus-scaledobject
  namespace: default
  labels:
    deploymentName: go-prom-app
spec:
  scaleTargetRef:
    deploymentName: go-prom-app
  pollingInterval: 15  # Optional. Default: 30 seconds
  cooldownPeriod:  30 # Optional. Default: 300 seconds
  minReplicaCount: 1   # Optional. Default: 0
  maxReplicaCount: 10 # Optional. Default: 100
  triggers:
  - type: prometheus
    metadata:
      # Required
      serverAddress: http://prometheus-service.default.svc.cluster.local:9090
      metricName: access_frequency
      threshold: '3'
      query: sum(rate(http_requests[2m]))
```

## Testing KEDA

### Make the demo app accessible locally
`kubectl port-forward service/go-prom-app-service 8080`

### View current pods
```
# kubectl get pod
NAME                                               READY   STATUS    RESTARTS   AGE
go-prom-app-5547b97ccd-l5p8v                       1/1     Running   0          6m
```

watch pod changes:
+ `kubectl get pods -l=app=go-prom-app -w`

### Start load testing
```
echo "GET http://localhost:8080/test" | vegeta attack -duration=100s | tee results.bin | vegeta report
```
I am using [vegeta](https://github.com/tsenart/vegeta) for the loading here.

You will see number of pods increases.
```
# kubectl get pods -l=app=go-prom-app -w

NAME                           READY   STATUS    RESTARTS   AGE
go-prom-app-5547b97ccd-l5p8v   1/1     Running   0          7h6m
go-prom-app-5547b97ccd-l8nsz   0/1     Pending   0          0s
go-prom-app-5547b97ccd-l8nsz   0/1     Pending   0          0s
go-prom-app-5547b97ccd-l8nsz   0/1     ContainerCreating   0          0s
go-prom-app-5547b97ccd-l8nsz   1/1     Running             0          4s
go-prom-app-5547b97ccd-7c64n   0/1     Pending             0          0s
go-prom-app-5547b97ccd-7c64n   0/1     Pending             0          0s
go-prom-app-5547b97ccd-tj8gw   0/1     Pending             0          0s
go-prom-app-5547b97ccd-tj8gw   0/1     Pending             0          0s
go-prom-app-5547b97ccd-7c64n   0/1     ContainerCreating   0          0s
go-prom-app-5547b97ccd-tj8gw   0/1     ContainerCreating   0          0s
```


