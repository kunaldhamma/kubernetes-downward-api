# Kubernetes Downward API

![image](https://user-images.githubusercontent.com/18049790/43352583-0b37edda-9269-11e8-9695-1e8de81acb76.png)

## Disclaimer

- Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
- Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only.

## Table of Contents

1. Introduction

- 1.1 Kubernetes Downward API

## 1. Introduction

### 1.1 Kubernetes Downward API

- The Downward API enables you to expose the pod’s own metadata to the processes running inside that pod.
- Currently, it allows you to pass the following information to your containers:
  - The pod’s name
  - The pod’s IP address
  - The namespace the pod belongs to
  - The name of the node the pod is running on
  - The name of the service account the pod is running under
  - The CPU and memory requests for each container
  - The CPU and memory limits for each container
  - The pod’s labels
  - The pod’s annotations

```
apiVersion: v1
kind: Pod
metadata:
  name: downward
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 16M
      limits:
        cpu: 100m
        memory: 128M
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          fieldPath: metadata.name
    - name: POD_NAMESPACE
          valueFrom:
        fieldRef:
          fieldPath: metadata.namespace
    - name: POD_IP
      valueFrom:
        fieldRef:
          fieldPath: status.podIP
    - name: NODE_NAME
      valueFrom:
        fieldRef:
          fieldPath: spec.nodeName
    - name: SERVICE_ACCOUNT
      valueFrom:
        fieldRef:
          fieldPath: spec.serviceAccountName
    - name: CONTAINER_CPU_REQUEST_MILLICORES
      valueFrom:
        resourceFieldRef:
          resource: requests.cpu
          divisor: 1m
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Ki
```

`k exec -it pod/downward sh`

```
/ # env
POD_IP=10.244.0.39
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.245.0.1:443
HOSTNAME=downward
SHLVL=1
HOME=/root
NODE_NAME=digital-ocean-pool-85nn8
CONTAINER_MEMORY_LIMIT_KIBIBYTES=125000
TERM=xterm
POD_NAME=downward
KUBERNETES_PORT_443_TCP_ADDR=10.245.0.1
SERVICE_ACCOUNT=default
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
CONTAINER_CPU_REQUEST_MILLICORES=15
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.245.0.1:443
POD_NAMESPACE=ckad
KUBERNETES_SERVICE_HOST=10.245.0.1
```

_End of Section_
