# Learning Kubernetes Ingress  

![Kubernetes](https://img.shields.io/badge/Kubernetes-326ce5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-FFCB2B?style=for-the-badge&logo=kubernetes&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Ingress](https://img.shields.io/badge/Ingress-NGINX-339933?style=for-the-badge&logo=nginx&logoColor=white)

---

````markdown
# Kubernetes Hands-On: Django App with Deployment, Service, and Ingress

This repository documents my learning journey with Kubernetes resources while deploying a simple Django-based Python web application. I go step by step: building a Docker image, deploying it on Kubernetes, exposing it via a Service, and finally accessing it using an Ingress resource with a custom domain.
````
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
<img width="2934" height="146" alt="image" src="https://github.com/user-attachments/assets/c291b013-ff32-48af-9f92-426d70d4c55c" />

---

### 5.1: Enable NGINX Ingress Controller

```bash
minikube addons enable ingress
```
<img width="2934" height="412" alt="image" src="https://github.com/user-attachments/assets/d001ec68-3de5-4e19-910b-1a2dbf9027f6" />


Check pods:

```bash
kubectl get pods -A | grep nginx
```
<img width="2934" height="186" alt="image" src="https://github.com/user-attachments/assets/7b0f99c2-8bf7-4d57-bc57-d9ac599b7128" />


* Controller runs in namespace `ingress-nginx`.
* Check logs to confirm the ingress resource is synced:

```bash
kubectl logs <nginx-ingress-pod-name> -n ingress-nginx
```
<img width="2934" height="186" alt="image" src="https://github.com/user-attachments/assets/8c87af54-367d-4b5e-b1c7-a58a46184500" />


---

### 5.2: Access via Ingress

After syncing, the ingress IP will be populated:

```bash
kubectl get ing
```
<img width="2934" height="142" alt="image" src="https://github.com/user-attachments/assets/c046bb48-0cfb-4f28-9878-9e9e43e332ee" />


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
