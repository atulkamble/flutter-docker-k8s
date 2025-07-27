a **complete guide** to create a Flutter app, **containerize it with Docker**, and **deploy it on Kubernetes**.

---

## 🚀 Project Structure Overview

```
flutter-k8s-app/
├── flutter_app/           # Your Flutter web app
├── Dockerfile             # Docker image build file
├── k8s/
│   ├── deployment.yaml    # K8s deployment
│   ├── service.yaml       # K8s service
│   └── ingress.yaml       # Optional: ingress
└── README.md              # Documentation
```

---

## 🧱 Step 1: Create a Flutter Web App
Here's a **complete minimal Flutter web app** you can use for the `flutter-k8s-webapp` project. This includes the full Flutter project with a basic homepage, ready to build for web and deploy using Docker + Kubernetes.

---

## 📁 Project Folder: `flutter_app/`

### Step-by-Step to Generate

If you're building from scratch:

```bash
flutter create flutter_app
cd flutter_app
```

---

### ✅ Updated Code: `lib/main.dart`

Replace the default content with:

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter K8s App',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const HomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('🚀 Flutter on Kubernetes'),
      ),
      body: const Center(
        child: Text(
          'Hello from Flutter running in Docker on Kubernetes!',
          textAlign: TextAlign.center,
          style: TextStyle(fontSize: 22),
        ),
      ),
    );
  }
}
```

---

### ⚙️ Web App Build

Run this to generate web files:

```bash
flutter build web
```

It will create output in:

```
flutter_app/build/web/
```

This will be copied in your Docker image and served using NGINX.

---

## 📝 Optional Updates

* In `pubspec.yaml`, no extra packages are required for this basic app.
* You can style it further using `GoogleFonts`, `Lottie`, etc. if needed.
---

## 🐳 Step 2: Dockerize Flutter Web App

### At root (`flutter-k8s-app/`), create `Dockerfile`:

```dockerfile
# Use Nginx as base image to serve the web app
FROM nginx:alpine

# Remove default nginx static files
RUN rm -rf /usr/share/nginx/html/*

# Copy Flutter web build to Nginx web directory
COPY flutter_app/build/web /usr/share/nginx/html

# Copy custom nginx config if needed (optional)
# COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

## 🔧 Step 3: Build & Run Locally with Docker

```bash
docker build -t flutter-web-app .
docker run -d -p 8080:80 flutter-web-app
```

Navigate to: [http://localhost:8080](http://localhost:8080)

---

## ☸️ Step 4: Kubernetes Deployment

### `k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flutter-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flutter-app
  template:
    metadata:
      labels:
        app: flutter-app
    spec:
      containers:
      - name: flutter-web
        image: <your-dockerhub-username>/flutter-web-app:latest
        ports:
        - containerPort: 80
```

### `k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flutter-app-service
spec:
  selector:
    app: flutter-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer  # Use NodePort for Minikube/local
```

---

## 🌐 (Optional) Ingress for Domain Mapping

### `k8s/ingress.yaml`

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flutter-app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: flutter.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: flutter-app-service
                port:
                  number: 80
```

> ⚠️ Requires NGINX Ingress Controller setup and DNS or local `/etc/hosts` entry for `flutter.local`.

---

## 📦 Step 5: Push Image to Docker Hub or ECR

```bash
docker tag flutter-web-app <dockerhub-username>/flutter-web-app:latest
docker push <dockerhub-username>/flutter-web-app:latest
```

---

## 🚀 Step 6: Deploy on Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc flutter-app-service
```

> Access via LoadBalancer/NodePort/Ingress depending on setup.

---

## ✅ Verify

```bash
kubectl get pods
kubectl get svc
kubectl describe svc flutter-app-service
```

---

## 📘 Sample README.md (for GitHub)

````markdown
# Flutter Web + Docker + Kubernetes

This project demonstrates how to:

- Build a Flutter Web App
- Dockerize the app with NGINX
- Deploy on Kubernetes cluster

## Quick Start

```bash
flutter build web
docker build -t flutter-web-app .
docker run -p 8080:80 flutter-web-app
````

## Kubernetes

```bash
kubectl apply -f k8s/
```
---
