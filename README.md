# Learning Kubernetes Services  

![Kubernetes](https://img.shields.io/badge/Kubernetes-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-FFCB2B?style=for-the-badge&logo=kubernetes&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)

---

````markdown
# Kubernetes Hands-On: Django App with Deployment, Service, and Ingress

This repository is designed to help beginners learn **Kubernetes resources** by deploying a simple Django-based Python web application.  
We will go step by step, starting from building a Docker image, deploying it on Kubernetes, exposing it via a Service, and finally accessing it using an Ingress resource with a custom domain.

---

## 📂 Repository Structure

Inside the `python-web-app` folder:

- **`devops/`** → The Django application code.  
- **`requirements.txt`** → Python dependencies.  
- **`Dockerfile`** → Instructions to build the Docker image.  
- **`deployment.yaml`** → Kubernetes Deployment manifest to run the app.  
- **`service.yaml`** → Kubernetes Service manifest to expose the app.  

> All manifests are ready to use, so you can follow along without creating new files.

---

## Step 1: Build the Docker Image

Our Dockerfile is simple:  

- Base image: **Ubuntu**  
- Installs **Python3, pip, and virtualenv**  
- Copies application code and requirements  
- Creates a virtual environment and installs dependencies  
- Exposes **port 8000** and runs Django automatically  

Build the Docker image:

```bash
cd python-web-app
docker build -t <docker-username>/python-sample-app-demo:v1 .
````

---

## Step 2: Push Image to Docker Hub

Kubernetes cannot pull images from your local machine. Push the image to Docker Hub:

```bash
docker push <docker-username>/python-sample-app-demo:v1
```

---

## Step 3: Deploy the Application

Apply the existing Deployment:

```bash
kubectl apply -f deployment.yaml
```

Check pods:

```bash
kubectl get pods -o wide
```

**Notes:**

* **Replicas:** 2 pods will be created.
* **Labels:** `app: sample-python-app` link the Deployment to the Service.
* **Ports:** 8000, as defined in Dockerfile.

---

## Step 4: Expose the App via Service

Apply the existing Service:

```bash
kubectl apply -f service.yaml
```

Check services:

```bash
kubectl get svc
```

Access options:

1. **Inside cluster:**

```bash
curl -L http://<cluster-ip>:80/demo
```

2. **NodePort (external):**

```bash
minikube ip
curl -L http://<minikube-ip>:30007/demo
```

3. **Port Forwarding (recommended):**

```bash
kubectl port-forward svc/python-django-app-service 8000:80
```

Then open in browser:

```
http://localhost:8000/demo
```

---

## Step 5: Access the App via Ingress

Ingress allows external traffic routing by **host and path**.

Create `ingress.yaml`:

Apply the Ingress:

```bash
kubectl apply -f ingress.yaml
kubectl get ing
```

> Initially, the **ADDRESS field is empty** because we need an Ingress Controller.

---

### 5.1: Enable NGINX Ingress Controller

```bash
minikube addons enable ingress
```

Check pods:

```bash
kubectl get pods -A | grep nginx
```

* Controller runs in namespace `ingress-nginx`.
* Check logs to confirm the ingress resource is synced:

```bash
kubectl logs <nginx-ingress-pod-name> -n ingress-nginx
```

---

### 5.2: Access via Ingress

After syncing, the ingress IP will be populated:

```bash
kubectl get ing
```

* Local testing requires mapping domain to IP in `/etc/hosts`:

```
192.168.49.2 foo.bar.com
```

* Then access:

```
http://foo.bar.com/bar
```

> In production, real domains would be used; here we mock it locally.

---

## 🎯 Key Learnings

* Deployments manage multiple pods with replicas.
* Labels and selectors link Deployments and Services.
* Services expose pods internally and externally.
* Ingress enables domain/path-based routing.
* For local clusters, `/etc/hosts` can mock domains.
* `curl -L` follows redirects, useful for Django apps.

---

```