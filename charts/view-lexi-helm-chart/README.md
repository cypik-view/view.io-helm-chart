# view-lexi Helm Chart

## Overview
This Helm chart deploys the `view-lexi` application with configurable parameters for image, service, persistence, environment variables, and ingress.

## Installation
To install the chart with the release name `test1` in the namespace `test`:

```sh
helm install test1 . -n test
```


## Configuration
The chart is configurable via the `values.yaml` file. Below are the available parameters:

### Image
```yaml
image:
  repository: "viewio/view-lexi"
  tag: "v1.1.0"
  pullPolicy: "IfNotPresent"
```

### Replica Count
```yaml
replicaCount: 1
```

### Service
```yaml
service:
  name: "lexi-service"
  type: "ClusterIP"
  port: 8201
  targetPort: 8201
```

### Persistence
```yaml
persistence:
  enabled: true
  storageClass: "standard"
  accessModes:
    - "ReadWriteMany"
  size: "5Gi"
  claimName: "lexi-shared-pvc"
```

### Environment Variables
```yaml
env:
  VIEW_ACCOUNT_GUID: "00000000-0000-0000-0000-000000000000"
  VIEW_DATABASE_TYPE: "Mysql"
  VIEW_DATABASE_HOST: "mysql"
  VIEW_DATABASE_PORT: 3306
  VIEW_DATABASE_NAME: "view"
  VIEW_DATABASE_USER: "root"
  VIEW_DATABASE_PASS: "password"
  VIEW_CONSOLE_LOGGING: "1"
  VIEW_CONTROL_HOSTNAME: "control.view.io"
  VIEW_CONTROL_PORT: 8403
  VIEW_CONTROL_SSL: "true"
  VIEW_CONTROL_SWVERSION: "v1.0.0"
  TERM: "xterm-256color"
```

### Volumes
```yaml
volumes:
  sharedStorage: "lexi-shared-pvc"
  mountPaths:
    - path: "/app/config"
      subPath: "view.json"
    - path: "/app/assets"
    - path: "/app/logs"
    - path: "/app/sourcedocs/default"
      subPath: "default"
    - path: "/app/temp"
    - path: "/app/ingest"
    - path: "/app/webhookreq"
    - path: "/app/webhookresp"
    - path: "/app/controllog"
    - path: "/app/controlapi"
    - path: "/app/controltoken"
```

## Accessing the Application
After installation, use the following commands based on the service type:

### Ingress Enabled
```sh
{{- if .Values.ingress.enabled }}
kubectl get ingress -n test
{{- end }}
```

### NodePort Service
```sh
export NODE_PORT=$(kubectl get --namespace test -o jsonpath="{.spec.ports[0].nodePort}" services test1)
export NODE_IP=$(kubectl get nodes --namespace test -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT
```

### LoadBalancer Service
```sh
kubectl get svc --namespace test test1
```

### ClusterIP Service
```sh
export POD_NAME=$(kubectl get pods --namespace test -l "app.kubernetes.io/name=test1" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace test port-forward $POD_NAME 8080:8201
```
Then visit `http://127.0.0.1:8080`

## Uninstallation
To uninstall the release:
```sh
helm uninstall test1 -n test
```

