## Deployment (most common)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-dep
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: nginx
        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"

        volumeMounts:
        - name: app-storage
          mountPath: /usr/share/nginx/html

      volumes:
      - name: app-storage
        persistentVolumeClaim:
          claimName: my-pvc
```
----------
## Pod (rare directly)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: nginx
```

----------

# 🌐 2. Networking

## Service (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 80
```

----------

## Service (NodePort)

```yaml
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30007
```

----------

## Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ing
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

----------

# 🔐 3. Config & Secrets

## ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  PORT: "3000"
```

----------

## Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  PASSWORD: mypass
```

----------

# 🔑 4. RBAC (security)

## Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: viewer
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","watch"]
```

----------

## RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: viewer-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: user
  namespace: dev
roleRef:
  kind: Role
  name: viewer
  apiGroup: rbac.authorization.k8s.io
```

----------

## ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: user
  namespace: dev
```

----------

# 💾 5. Storage

## PersistentVolume (manual)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

----------

## PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

----------

## StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local
provisioner: rancher.io/local-path
```

----------

# 📊 6. Monitoring (Prometheus stack)

## ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: myapp
  namespaceSelector:
    matchNames:
    - default
  endpoints:
  - port: http
    path: /metrics
```

----------

## PrometheusRule (alerts)

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: alert-rule
  namespace: monitoring
spec:
  groups:
  - name: alerts
    rules:
    - alert: PodCrashLoop
      expr: kube_pod_container_status_waiting_reason{reason="CrashLoopBackOff"} == 1
      for: 2m
      labels:
        severity: critical
      annotations:
        summary: "Pod in CrashLoopBackOff"
        description: "Pod {{ $labels.pod }}"
```

----------

# 📈 7. Autoscaling

## HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app-dep
  minReplicas: 2
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

----------

# 📦 8. Namespace (isolation)

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

----------

# ⚙️ 9. Job (batch work)

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-demo
spec:
  template:
    spec:
      containers:
      - name: job
        image: busybox
        command: ["echo","hello"]
      restartPolicy: Never
```

----------

# 🧠 Final reality check
