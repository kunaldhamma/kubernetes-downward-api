# Kubernetes Downward API

![image](https://user-images.githubusercontent.com/18049790/43352583-0b37edda-9269-11e8-9695-1e8de81acb76.png)

## Disclaimer

- Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
- Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only.

## Table of Contents

1. Introduction

- 1.1 Kubernetes Downward API
- 1.2 Exposing metadata through environment variables
- 1.3 Exposing metadata though volumes

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

### 1.2 Exposing metadata through environment variables

Sample YAML to showcase exposing pod metadata via env: downward-api-env.yaml

To test this grab the file here: `wget https://raw.githubusercontent.com/jamesbuckett/kubernetes-downward-api/master/downward-api-env.yaml`

```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-env
spec:
  containers:
  - name: main
    image: busybox
    command: ["sleep", "9999999"]
    resources:
      requests:
        cpu: 15m
        memory: 128M
      limits:
        cpu: 100m
        memory: 256M
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

Sample output after running downward-api-env.yaml:

```bash
`kubectl exec pod/downward-env env`
```

```bash
root@digital-ocean-droplet:~# kubectl exec -it pod/downward-env env
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=downward-env
NODE_NAME=digital-ocean-pool-xxxxx - The name of the node the pod is running on
SERVICE_ACCOUNT=default - The name of the service account
CONTAINER_CPU_REQUEST_MILLICORES=15 - The CPU and memory requests for each container
CONTAINER_MEMORY_LIMIT_KIBIBYTES=250000 - The CPU and memory limits for each container
POD_NAME=downward-env - The pod’s name
POD_NAMESPACE=ckad - The namespace the pod belongs to
POD_IP=10.244.0.1 - The pod’s IP address
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.245.0.1
KUBERNETES_SERVICE_HOST=10.245.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.245.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.245.0.1:443
TERM=xterm
HOME=/root
```

### 1.3 Exposing metadata through volumes

Sample YAML to showcase exposing pod metadata via env: downward-api-vol.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: downward-vol
  labels:
    foo: bar
  annotations:
    key1: value1
    key2: |
      multi
      line
      value
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
      volumeMounts:
        - name: downward
          mountPath: /etc/downward
  volumes:
    - name: downward
      downwardAPI:
        items:
          - path: "podName"
            fieldRef:
              fieldPath: metadata.name
          - path: "podNamespace"
            fieldRef:
              fieldPath: metadata.namespace
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "annotations"
            fieldRef:
              fieldPath: metadata.annotations
          - path: "containerCpuRequestMilliCores"
            resourceFieldRef:
              containerName: main
              resource: requests.cpu
              divisor: 1m
          - path: "containerMemoryLimitBytes"
            resourceFieldRef:
              containerName: main
              resource: limits.memory
              divisor: 1
```

To test this grab the file here: `wget https://raw.githubusercontent.com/jamesbuckett/kubernetes-downward-api/master/downward-api-env.yaml`

```bash
kubectl exec -it pod/downward-vol sh
```

```bash
root@digital-ocean-droplet:~# k exec -it pod/downward-vol sh
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.

/ # ls -la
total 44
drwxr-xr-x    1 root     root          4096 Aug  9 11:51 .
drwxr-xr-x    1 root     root          4096 Aug  9 11:51 ..
drwxr-xr-x    2 root     root         12288 Jun  7 17:34 bin
drwxr-xr-x    5 root     root           360 Aug  9 11:51 dev
drwxr-xr-x    1 root     root          4096 Aug  9 11:51 etc
drwxr-xr-x    2 nobody   nobody        4096 Jun  7 17:34 home
dr-xr-xr-x  194 root     root             0 Aug  9 11:51 proc
drwx------    1 root     root          4096 Aug  9 11:53 root
dr-xr-xr-x   13 root     root             0 Aug  9 11:51 sys
drwxrwxrwt    2 root     root          4096 Jun  7 17:34 tmp
drwxr-xr-x    3 root     root          4096 Jun  7 17:34 usr
drwxr-xr-x    1 root     root          4096 Aug  9 11:51 var

/ # cd etc/downward/

/etc/downward # ls -la
total 4
drwxrwxrwt    3 root     root           200 Aug  9 11:51 .
drwxr-xr-x    1 root     root          4096 Aug  9 11:51 ..
drwxr-xr-x    2 root     root           160 Aug  9 11:51 ..2021_08_09_11_51_16.095792186
lrwxrwxrwx    1 root     root            31 Aug  9 11:51 ..data -> ..2021_08_09_11_51_16.095792186
lrwxrwxrwx    1 root     root            18 Aug  9 11:51 annotations -> ..data/annotations
lrwxrwxrwx    1 root     root            36 Aug  9 11:51 containerCpuRequestMilliCores -> ..data/containerCpuRequestMilliCores
lrwxrwxrwx    1 root     root            32 Aug  9 11:51 containerMemoryLimitBytes -> ..data/containerMemoryLimitBytes
lrwxrwxrwx    1 root     root            13 Aug  9 11:51 labels -> ..data/labels
lrwxrwxrwx    1 root     root            14 Aug  9 11:51 podName -> ..data/podName
lrwxrwxrwx    1 root     root            19 Aug  9 11:51 podNamespace -> ..data/podNamespace

/etc/downward # cat podName
downward-vol - The pod’s name
```

_End of Section_
