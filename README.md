# Kubernetes Downward API 

![image](https://user-images.githubusercontent.com/18049790/43352583-0b37edda-9269-11e8-9695-1e8de81acb76.png)

## Disclaimer
* Opinions expressed here are solely my own and do not express the views or opinions of JPMorgan Chase.
* Any third-party trademarks are the intellectual property of their respective owners and any mention herein is for referential purposes only. 

## Table of Contents

1. Introduction
* 1.1 Kubernetes Downward API

## 1. Introduction

### 1.1 Kubernetes Downward API
* The Downward API enables you to expose the pod’s own metadata to the processes running inside that pod. 
* Currently, it allows you to pass the following information to your containers:
  * The pod’s name
  * The pod’s IP address
  * The namespace the pod belongs to
  * The name of the node the pod is running on
  * The name of the service account the pod is running under
  * The CPU and memory requests for each container
  * The CPU and memory limits for each container
  * The pod’s labels
  * The pod’s annotations

'''
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
        memory: 100Ki
      limits:
        cpu: 100m
        memory: 4Mi
    env:
    - name: POD_NAME
      valueFrom:                                   ❶
        fieldRef:                                  ❶
          fieldPath: metadata.name                 ❶
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
      valueFrom:                                   ❷
        resourceFieldRef:                          ❷
          resource: requests.cpu                   ❷
          divisor: 1m                              ❸
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom:
        resourceFieldRef:
          resource: limits.memory
          divisor: 1Ki
'''








*End of Section*
