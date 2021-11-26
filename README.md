
Deploying Yellowfin For Squick


1. Installation.yaml for yellowfin

```
### Yellowfin Cluster Service ###
apiVersion: v1
kind: Service
metadata:
  name: yellowfin-cluster
 
spec:
  ports:
    - protocol: TCP
      name: web
      port: 8080
  selector:
    app: yellowfin-cluster
---
### Yellowfin Cluster Deployment ###
kind: Deployment
apiVersion: apps/v1
metadata:
  name: yellowfin-cluster
  labels:
    app: yellowfin-cluster
 
spec:
  replicas: 1
  selector:
    matchLabels:
      app: yellowfin-cluster
  template:
    metadata:
      labels:
        app: yellowfin-cluster
    spec:
      setHostnameAsFQDN: true
      #hostname: reports.tst-01.eu-west-3.servicequik.com
      containers:
        - env:
          - name: APP_MEMORY
            value: "6144"
          - name: CLUSTER_PORT
            value: "7800"
          - name: JDBC_CLASS_NAME
            value: org.postgresql.Driver
          - name: JDBC_CONN_ENCRYPTED
            value: "false"
          - name: JDBC_CONN_PASS
            value: "Your-password-here"
          - name: JDBC_CONN_URL
            value: "jdbc:postgresql://prdsqdb-yellowfin.ckk5swrowe4b.eu-west-1.rds.amazonaws.com:5432/yellowfindb"
          - name: JDBC_CONN_USER
            value: "postgres"
          - name: PROXY_HOST
            value: "reports.zing.work"
          - name: PROXY_PORT
            value: "80"
          - name: PROXY_SCHEME
            value: "http"
          - name: HOSTNAME
            value: "reports.zing.work"
          name: yellowfin-cluster
          #image: yellowfinbi/yellowfin-all-in-one:9.6
          securityContext:
            privileged: true
            #image: yellowfinbi/yellowfin-app-only:9.6
          image: 065409082096.dkr.ecr.ap-southeast-1.amazonaws.com/servicequik-yellowfin-reporting-service:3c13e3f
          ports:
            - name: web
              containerPort: 8080
    

```
2. Deploying  the installations

            $ kubectl apply -f installation.yaml


3. Deploying the ingress controller :

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.0/deploy/static/provider/cloud/deploy.yaml

4. Ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: yellowfin-ingress
  namespace: yellowfin
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.org/proxy-connect-timeout: "30s"
    nginx.org/proxy-read-timeout: "20s"
    nginx.org/client-max-body-size: "4m"
    nginx.ingress.kubernetes.io/configuration-snippet: |
      more_set_input_headers Host reports.tst-01.eu-west-3.servicequik.com;
    nginx.ingress.kubernetes.io/use-regex: "true"
spec:
  rules:
  - host: reports.zing.work
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: yellowfin-cluster
            port:
              number: 8080
```

5. Deploying the ingress

$ kubectl apply -f ingress.yaml

6. Changing the hostname on the pod , so that the yellowfin application reflects the same.
![image](https://user-images.githubusercontent.com/86567992/143531617-bd1daef8-73ff-4021-923d-88605bedd799.png)

7.Accessing through the hostname ; reports.zing.work

![yfin](https://user-images.githubusercontent.com/86567992/143531725-aa01b1bc-0ca5-487b-a9e1-88951af8c520.png)

