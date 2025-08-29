# Kubernetes basic understanding and working

# 🌐 Introduction to Kubernetes


## 🔹 What is Kubernetes?
Kubernetes (often abbreviated as **K8s**) is an **open-source container orchestration platform** developed by **Google** and now maintained by the **Cloud Native Computing Foundation (CNCF)**.  
It automates the **deployment, scaling, and management** of containerized applications.


## 🔹 Why Kubernetes?
Managing a few containers manually with **Docker** is easy, but in real-world applications where there are **hundreds or thousands of containers**, it becomes complex.  
Kubernetes helps by:

- Automatically handling **deployment** of containers.  
- Ensuring **high availability** with self-healing.  
- **Scaling up/down** applications based on demand.  
- Providing **service discovery & load balancing**.  
- Managing **storage and configurations**.  


## 🔹 Key Features
- **Automated Rollouts & Rollbacks** – update applications without downtime.  
- **Self-Healing** – restarts failed containers, replaces unhealthy ones.  
- **Horizontal Scaling** – scale applications up or down easily.  
- **Load Balancing & Service Discovery** – distribute traffic automatically.  
- **Storage Orchestration** – automatically mount storage (local, cloud, etc.).  
- **Configuration & Secrets Management** – manage app configs securely.  

## Steps:
1. Firstly download Kubernetes and minikube for windows

2. To run:
Firstly we need to build an image so go to docker and check nginx official image

Note: to build an image minikube should be running execute > minikube start

3. use the command ->  kubectl create deployment my-nginx --image=nginx:latest  //my-nginx is variable name here any name can be given
   
> kubectl get deployments

> kubectl get pods

execute ->  minikube addons enable metrics-server
then -> minikube dashboard

to open dashboard locally in browser

> minikube status

> minikube delete  //to delete deployment


> kubectl expose deployment my-nginx --port=80 --type=LoadBalancer

> minikube service my-nginx



## Buidling docker image

> docker build -t sheikhjaveed3124/kubernetes-web-app:1.0 .

> docker push sheikhjaveed3124/kubernetes-web-app:1.0      

> kubectl delete my-demoapp


## Rollout and Rollback

> kubectl set image deployment kubernetes-demo-app kubernetes-web-app=sheikhjaveed3124/kubernetes-web-app:05

> kubectl rollout status deployment kubernetes-demo-app
      
> kubectl rollout undo deployment kubernetes-demo-app  


## Self-healing in K8s
in localhost:3000 if we type -> localhost:3000/exit -> it will stop the server

run this as a service in Kubernetes and while running we enter /exit path -> in dashboard see the pod shows red color indicating the crash of the website but within few seconds it restarts the service on its own again it becomes green on dashboard

> kubectl create deployment kubernetes-node-demo --image=philippaul/node-demo-app:01


> kubectl expose deployment kubernetes-node-demo --type=LoadBalancer --port=3000

> minikube service kubernetes-node-demo


### deployment using yaml file 


> kubectl apply -f .\deployment-node-app.yml
> kubectl get deployments


deployment-node-app.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: kubernetes-node-demo
spec:
  # 2 Pods should exist at all times.
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: node-app
    spec:
      containers:
      - name: node-app
        # Run this image
        image: philippaul/node-demo-app:01
```

### service using yml file

> kubectl get services

> kubectl apply -f .\service-node-app.yml


service-node-app.yml
```bash
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: service-my-node-app
spec:
  ports:
    # Accept traffic sent to port 8080
    - name: http
      port: 8080
      targetPort: 3000
  selector:
    # Loadbalance traffic across Pods matching
    # this label selector
    app: node-app
  # Create an HA proxy in the cloud provider
  # with an External IP address - *Only supported
  # by some cloud providers*
  type: LoadBalancer
```

## Web app with mongo connection example

> docker run -p 27017:27017 -d --name mongodb mongo

> docker run --network my-net -p 3000:3000 --name sheikhjaveed3124/demo-mongodb:01


## Running Multiple containers

1. Multiple containers in single pod

> kubectl get deployments

> kubectl apply -f .\deployment-node-app.yml


> minikube service service-my-nodedb-app


| NAMESPACE |         NAME          | TARGET PORT |            URL            |
|-----------|-----------------------|-------------|---------------------------|
| default   | service-my-nodedb-app | http/8080   | http://192.168.49.2:31848 |

🏃  Starting tunnel for service service-my-nodedb-app.


| NAMESPACE |         NAME          | TARGET PORT |          URL           |
|-----------|-----------------------|-------------|------------------------|
| default   | service-my-nodedb-app |             | http://127.0.0.1:54145 |


🎉  Opening service default/service-my-nodedb-app in default browser...
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.


deployment-node-app.yml

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: kubernetes-node-mongodb
spec:
  # 2 Pods should exist at all times.
  replicas: 2
  selector:
    matchLabels:
      app: nodedb-app
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: nodedb-app
    spec:
      containers:
      - name: nodedb-app
        # Run this image
        image: sheikhjaveed3124/demo-mongodb:02
      - name: mongodb
        image: mongo:latest
```

service-node-app.yml

```bash
apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: service-my-nodedb-app
spec:
  ports:
    # Accept traffic sent to port 8080
    - name: http
      port: 8080
      targetPort: 3000
  selector:
    # Loadbalance traffic across Pods matching
    # this label selector
    app: nodedb-app
  # Create an HA proxy in the cloud provider
  # with an External IP address - *Only supported
  # by some cloud providers*
  type: LoadBalancer
```

### combined deployment and service file
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: kubernetes-node-mongodb
spec:
  # 2 Pods should exist at all times.
  replicas: 2
  selector:
    matchLabels:
      app: nodedb-app
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: nodedb-app
    spec:
      containers:
      - name: nodedb-app
        # Run this image
        image: sheikhjaveed3124/demo-mongodb:02
      - name: mongodb
        image: mongo:latest

---

apiVersion: v1
kind: Service
metadata:
  # Unique key of the Service instance
  name: service-my-nodedb-app
spec:
  ports:
    # Accept traffic sent to port 8080
    - name: http
      port: 8080
      targetPort: 3000
  selector:
    # Loadbalance traffic across Pods matching
    # this label selector
    app: nodedb-app
  # Create an HA proxy in the cloud provider
  # with an External IP address - *Only supported
  # by some cloud providers*
  type: LoadBalancer
```

----------

mongo-app.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  # Unique key of the Deployment instance
  name: mongo-app-diff-pod
spec:
  # 2 Pods should exist at all times.
  replicas: 1
  selector:
    matchLabels:
      app: mongo-app-diff-pod
  template:
    metadata:
      labels:
        # Apply this label to pods and default
        # the Deployment label selector to this value
        app: mongo-app-diff-pod
    spec:
      containers:
      - name: mongo-app-diff-pod
        image: mongo:latest

---

apiVersion: v1
kind: Service
metadata:
  name: service-diff-mongo-app
spec:
  ports:
    - name: tcp
      port: 27017
      targetPort: 27017
  selector:
    app: mongo-app-diff-pod

  type: LoadBalancer
```

2. Multiple containers in separate pods

> kubectl apply -f .\mongo-config.yml

> kubectl apply -f .\node-app.yml
deployment.apps/node-app created
service/service-node-app created

> minikube service service-node-app      

| NAMESPACE |       NAME       | TARGET PORT |            URL            |
|-----------|------------------|-------------|---------------------------|
| default   | service-node-app | http/8080   | http://192.168.49.2:30506 |

🏃  Starting tunnel for service service-node-app.


| NAMESPACE |       NAME       | TARGET PORT |          URL           |
|-----------|------------------|-------------|------------------------|
| default   | service-node-app |             | http://127.0.0.1:61326 |

🎉  Opening service default/service-node-app in default browser...
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.
✋  Stopping tunnel for service service-node-app.

> minikube service service-diff-mongo-app

| NAMESPACE |          NAME          | TARGET PORT |            URL            |
|-----------|------------------------|-------------|---------------------------|
| default   | service-diff-mongo-app | tcp/27017   | http://192.168.49.2:30274 |

🏃  Starting tunnel for service service-diff-mongo-app.

| NAMESPACE |          NAME          | TARGET PORT |          URL           |
|-----------|------------------------|-------------|------------------------|
| default   | service-diff-mongo-app |             | http://127.0.0.1:61362 |


🎉  Opening service default/service-diff-mongo-app in default browser...
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.

> kubectl apply -f .\node-app.yml
deployment.apps/node-app configured

> minikube service service-diff-mongo-app


| NAMESPACE |          NAME          | TARGET PORT |            URL            |
|-----------|------------------------|-------------|---------------------------|
| default   | service-diff-mongo-app | tcp/27017   | http://192.168.49.2:30274 |

🏃  Starting tunnel for service service-diff-mongo-app.

| NAMESPACE |          NAME          | TARGET PORT |          URL           |
|-----------|------------------------|-------------|------------------------|
| default   | service-diff-mongo-app |             | http://127.0.0.1:58668 |

🎉  Opening service default/service-diff-mongo-app in default browser...
❗  Because you are using a Docker driver on windows, the terminal needs to be open to run it.

> kubectl apply -f .\mongo-config.yml
configmap/mongo-config unchanged

> kubectl apply -f .\node-app.yml
deployment.apps/node-app unchanged
service/service-node-app unchanged


mongo-db.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo-app-diff-pod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo-app-diff-pod
  template:
    metadata:
      labels:
        app: mongo-app-diff-pod
    spec:
      containers:
      - name: mongo-app-diff-pod
        image: mongo:latest
---

# MongoDB Service
apiVersion: v1
kind: Service
metadata:
  name: service-diff-mongo-app
spec:
  ports:
    - port: 27017
      targetPort: 27017
  selector:
    app: mongo-app-diff-pod
  type: ClusterIP   # keep internal only
```


node-app.yml
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: node-app
  template:
    metadata:
      labels:
        app: node-app
    spec:
      containers:
      - name: node-app
        image: sheikhjaveed3124/demo-mongodb:03
        envFrom:
          - configMapRef:
              name: mongo-config
```
---
# Node.js Service
```bash
apiVersion: v1
kind: Service
metadata:
  name: service-node-app
spec:
  ports:
    - name: http
      port: 8080
      targetPort: 3000
  selector:
    app: node-app
  type: LoadBalancer
```

mongo-config.yml
```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: mongo-config
data:
  MONGO_HOST: "service-diff-mongo-app"
  MONGO_PORT: "27017"
```


## Persistent Volumes

## 🔹 Definition
A **Persistent Volume (PV)** in Kubernetes is a piece of **storage in the cluster** that has been provisioned by an administrator or dynamically provisioned using **Storage Classes**.  
It is an **abstraction layer** over the physical storage (local disk, NFS, AWS EBS, GCP Persistent Disk, Azure Disk, etc.) and provides storage resources to Pods.

---

## 🔹 Why Persistent Volumes?
By default, when a Pod is deleted, its data is also lost because container storage is **ephemeral**.  
To store data **persistently** (databases, logs, files), Kubernetes uses **Persistent Volumes**.

host-pv.yml
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/
    type: DirectoryOrCreate
```

host-pvc.yml
```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  volumeName: host-pv
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```
