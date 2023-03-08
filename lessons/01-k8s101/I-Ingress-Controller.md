---
title: "Ingress Controller "
description: " kubernetes Service "
---

#### enable ingress addon 

```sh
k8s101 git:(main) ✗ minikube addons enable ingress
💡  ingress is an addon maintained by Kubernetes. For any concerns contact minikube on GitHub.
You can view the list of minikube maintainers at: https://github.com/kubernetes/minikube/blob/master/OWNERS
💡  After the addon is enabled, please run "minikube tunnel" and your ingress resources would be available at "127.0.0.1"
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
    ▪ Using image k8s.gcr.io/ingress-nginx/controller:v1.2.1
    ▪ Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
🔎  Verifying ingress addon...
🌟  The 'ingress' addon is enabled
➜  k8s101 git:(main) ✗ 
```
#### verify ngnix controller running 

```sh
➜  k8s101 git:(main) ✗ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-fxzbs        0/1     Completed   0          4m7s
ingress-nginx-admission-patch-jw98n         0/1     Completed   1          4m7s
ingress-nginx-controller-5959f988fd-tv8x8   1/1     Running     0          4m7s

```
#### verify all pods running 

```sh
➜  k8s101 git:(main) ✗ kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS      AGE
coredns-565d847f94-bl9qz           1/1     Running   0             12h
etcd-minikube                      1/1     Running   0             12h
kube-apiserver-minikube            1/1     Running   0             12h
kube-controller-manager-minikube   1/1     Running   0             12h
kube-proxy-qj7s7                   1/1     Running   0             12h
kube-scheduler-minikube            1/1     Running   0             12h
storage-provisioner                1/1     Running   2 (12h ago)   12h

```

#### Deploy Hello World App 

```sh

➜  k8s101 git:(main) ✗ kubectl create deployment web --image=gcr.io/google-samples/hello-app:1.0
deployment.apps/web created
```

```sh
➜  k8s101 git:(main) ✗ kubectl expose deployment web --type=NodePort --port=8080
service/web exposed
```

```sh
➜  k8s101 git:(main) ✗ kubectl get service web      
NAME   TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
web    NodePort   10.100.132.142   <none>        8080:30646/TCP   41s

```

```sh
➜  k8s101 git:(main) ✗ minikube service web --url
http://127.0.0.1:51575
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

output 
```
Hello, world!
Version: 1.0.0
Hostname: web-84fb9498c7-zx2k4
```

### Ingress that sends traffic to your Service via hello-world.info.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
  - host: hello-world.info
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 8080

```

```sh
➜  k8s101 git:(main) ✗ kubectl apply -f example-ingress.yaml
ingress.networking.k8s.io/example-ingress created
```

#### verify IP addree 

```sh
➜  k8s101 git:(main) ✗ kubectl get ingress
NAME              CLASS   HOSTS              ADDRESS        PORTS   AGE
example-ingress   nginx   hello-world.info   192.168.49.2   80      18m
➜  k8s101 git:(main) ✗ 
```


```sh
➜  k8s101 git:(main) ✗ echo "127.0.0.1  hello-world.info" | sudo tee -a /etc/hosts 
127.0.0.1  hello-world.info
➜  k8s101 git:(main) ✗ kubectl apply -f example-ingress.yaml                      
ingress.networking.k8s.io/example-ingress unchanged
```
#### add path 

```yaml
    - path: /v2
        pathType: Prefix
        backend:
          service:
            name: web2
            port:
              number: 8080

```

```sh
➜  k8s101 git:(main) ✗ kubectl apply -f example-ingress.yaml           
ingress.networking.k8s.io/example-ingress configured
```

```sh
➜  k8s101 git:(main) ✗ minikube tunnel
✅  Tunnel successfully started

📌  NOTE: Please do not close this terminal as this process must stay alive for the tunnel to be accessible ...

❗  The service/ingress example-ingress requires privileged ports to be exposed: [80 443]
🔑  sudo permission will be asked for it.
🏃  Starting tunnel for service example-ingress.

```

### check output 

```sh
http://hello-world.info/
http://hello-world.info/v2
```