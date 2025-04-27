Kubernetes Annotations Labs
This repository contains hands-on labs to demonstrate the use of Kubernetes Annotations in a Minikube cluster. The labs focus on practical scenarios relevant to a banking environment, such as tracking deployment metadata, integrating with monitoring tools, and configuring Ingress routing. The labs are designed to be beginner-friendly with step-by-step instructions.
Table of Contents

Overview
Prerequisites
Lab 1: Adding Annotations to a Deployment
Lab 2: Annotations for Prometheus Monitoring
Lab 3: Annotations for Ingress Configuration
Cleanup
Additional Resources

Overview
Kubernetes Annotations are key-value pairs used to attach non-identifying metadata to objects like Pods, Deployments, and Ingresses. Unlike labels, annotations are not used for querying but for integration with external tools (e.g., CI/CD pipelines, monitoring systems) or storing additional information (e.g., build metadata, owner details).
These labs simulate real-world banking use cases:

Lab 1: Store build metadata and owner info in a Deployment for a banking app.
Lab 2: Configure Prometheus monitoring annotations for a transaction API.
Lab 3: Use Ingress annotations to customize routing for a payment API.

Prerequisites
Before starting, ensure you have the following:

Minikube installed and running:minikube start
minikube status


kubectl configured to use the Minikube context:kubectl config current-context


A text editor (e.g., VS Code, nano) to create YAML files.
Basic familiarity with Kubernetes concepts (Pods, Deployments, Services, Ingress).

Lab 1: Adding Annotations to a Deployment
This lab demonstrates how to add annotations to a Deployment and its Pods to store build metadata and owner information, simulating version tracking for a banking application.
Steps

Create the Deployment YAML:Create a file named deployment-annotations.yaml with the following content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-app
  annotations:
    bank.com/build-id: "2025.04.27.001"
    bank.com/owner: "devops-team@bank.com"
    bank.com/version: "v1.2.3"
spec:
  replicas: 2
  selector:
    matchLabels:
      app: banking-app
  template:
    metadata:
      labels:
        app: banking-app
      annotations:
        bank.com/deploy-time: "2025-04-27T10:00:00Z"
        bank.com/debug: "enabled"
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80


Apply the Deployment:
kubectl apply -f deployment-annotations.yaml


Verify Annotations:

Check Deployment annotations:kubectl get deployment banking-app -o yaml | grep -A 3 annotations

Expected output includes bank.com/build-id, bank.com/owner, etc.
Check Pod annotations:kubectl get pod -l app=banking-app -o yaml | grep -A 2 annotations

Expected output includes bank.com/deploy-time, bank.com/debug.


Inspect Details:
kubectl describe deployment banking-app | grep -A 3 Annotations
kubectl describe pod -l app=banking-app | grep -A 2 Annotations



Lab 2: Annotations for Prometheus Monitoring
This lab shows how to add Prometheus-compatible annotations to a Pod to enable metrics scraping, simulating a transaction API in a banking environment.
Steps

Create the Deployment YAML:Create a file named prometheus-annotations.yaml with the following content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: transaction-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: transaction-api
  template:
    metadata:
      labels:
        app: transaction-api
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "80"
        prometheus.io/path: "/metrics"
        bank.com/service: "transaction-processing"
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80


Apply the Deployment:
kubectl apply -f prometheus-annotations.yaml


Verify Annotations:
kubectl get pod -l app=transaction-api -o yaml | grep -A 4 annotations

Expected output includes prometheus.io/scrape, prometheus.io/port, etc.

Inspect Details:
kubectl describe pod -l app=transaction-api | grep -A 4 Annotations

Note: This lab simulates Prometheus integration. If Prometheus is installed, it would detect these annotations for scraping.


Lab 3: Annotations for Ingress Configuration
This lab demonstrates how to use Ingress annotations to customize routing for a payment API, ensuring secure and correct API access in a banking environment.
Steps

Enable Ingress in Minikube:
minikube addons enable ingress


Create the Configuration YAML:Create a file named ingress-annotations.yaml with the following content:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment-api
spec:
  replicas: 1
  selector:
    matchLabels:
      app: payment-api
  template:
    metadata:
      labels:
        app: payment-api
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: payment-api-svc
spec:
  selector:
    app: payment-api
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: payment-api-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    bank.com/api-type: "payment"
spec:
  rules:
  - http:
      paths:
      - path: /payment
        pathType: Prefix
        backend:
          service:
            name: payment-api-svc
            port:
              number: 80


Apply the Configuration:
kubectl apply -f ingress-annotations.yaml


Verify Annotations:
kubectl get ingress payment-api-ingress -o yaml | grep -A 3 annotations

Expected output includes nginx.ingress.kubernetes.io/rewrite-target, bank.com/api-type, etc.

Test the Ingress:

Get the Minikube IP:minikube ip


Test the /payment path:curl http://<minikube-ip>/payment

Expected output: Nginx welcome page (or an SSL redirect error if enforced).



Cleanup
To remove all resources created in these labs:
kubectl delete -f deployment-annotations.yaml
kubectl delete -f prometheus-annotations.yaml
kubectl delete -f ingress-annotations.yaml

Additional Resources

Kubernetes Annotations Documentation
NGINX Ingress Controller Annotations
Minikube Documentation


Feel free to fork this repository, experiment with the labs, and contribute improvements!
