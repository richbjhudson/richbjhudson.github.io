---
layout: post
title:  "Linux - How to Deploy Containers to a Kubernetes Cluster"
date:   2023-02-17 13:30 +0000
categories: Linux
---
# Prerequisites
- [Setup a K8s Cluster]({{ site.baseurl }}/linux/2023/02/15/setup_k8s/)

# Steps - Performed on the Controller
## Create K8s Deployment YAML for a Pod and Service
- Create a directory to store the deployment files `mkdir k8s_services`
- Create a YAML file that describes the *Pod and Container* specification `sudo pod.yml`:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-example
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: linuxserver/nginx
      ports:
        - containerPort: 80
          name: "nginx-http"
```

*Note: Labels are key values pairs for reference. linuxserver/nginx is the https://linuxserver.io image repo that support both x86 and ARM architectures (Pis). containerPort is the port the container exposes to the internal K8s network only at this stage.*

- Create a YAML file that describes the *NodePort service* to provide the ability to map a network port on a Pod to the Node it is running on `sudo service-nodeport.yml`:
```
apiVersion: v1
kind: Service
metadata:
  name: nginx-example
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      nodePort: 30080
      targetPort: nginx-http
  selector:
    app: nginx
```
*Note: nodePort may be between 30000 - 32767. The selector value is a way to reference pods that the configuration applies to using the pod's label.*

## Apply the Deployment YAML to the K8s Cluster
- Apply the *Pod and Container* specification: `kubectl apply -f pod.yml`
  - Check the pod status `kubectl get pods`, you can see additional field with `kubectl get pods -o wide`.
- Apply the *NodePort service* specification: `kubectl apply -f service-nodeport.yml`
  - Check the status of the service `kubectl get services`
  - Test connectivity using:
  ```
  curl http://192.168.101.90:30080
  curl http://192.168.101.91:30080
  curl http://192.168.101.92:30080
  ```
  *Note: If you browse to any node ip address in the cluster on the given port the requests are directed to the node.*

## Remove the Pod and Service K8s Cluster
- Remove the *Pod and Container* specification:
```
kubectl delete pod nginx-example
kubectl get pods
```
- Remove the *NodePort service* specification:
```
kubectl delete service nginx-example
kubectl get services
```