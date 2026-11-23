# Week 11 – DevOps Lab: Containerization (Lab 11)

**Student:** Yigit Alkoc
**Student ID:** 101558073
**Course:** DevOps – George Brown College
**Lab:** Lab 11 – Containerization

This repository contains all the code for Week 11 DevOps Lab 11:

- **Part I** – Build a custom Docker image of a Python (Flask + Redis) web app using a `Dockerfile`, run it as a container, and push it to Docker Hub.
- **Part II** – Deploy an existing nginx image on Kubernetes (Minikube) using a Deployment and a NodePort Service.

---

## Repository Contents

```
week11-devops-lab/
├── README.md                  <- this file
├── lab11_Docker/
│   ├── Dockerfile             <- image definition (python:3.11-slim)
│   ├── app.py                 <- Flask web app (Hello <NAME> + Redis visit counter)
│   ├── requirements.txt       <- Flask, Redis
│   └── ReadMe.md              <- command list from the lab handout
└── kubernetes/
    ├── nginx-deploy.yaml      <- nginx Deployment (3 replicas)
    └── nginx-service.yaml     <- nginx NodePort Service (nodePort 30080)
```

The Kubernetes YAML files come from the instructor's exercise repository
(<https://github.com/pritamworld/DevOps_Kubernetes_exercise9>, `k8/` folder), which is
forked as part of Part II.

---

## Part I – Build and run the Docker image

All commands are run from inside the `lab11_Docker` folder.

### 0. Test the Docker installation

```bash
docker run hello-world
```

### 1. Build the image

```bash
docker build --tag=hellodocker .
```

The `Dockerfile`:

- starts from the official `python:3.11-slim` image,
- sets the working directory to `/app` and copies the project files into it,
- installs the dependencies from `requirements.txt` (`Flask`, `Redis`) with pip,
- exposes port **80**,
- sets the environment variable `NAME=World`,
- runs `python app.py` when the container starts.

### 2. Verify the image was created

```bash
docker image ls
```

### 3. Run the container

```bash
docker run -p 4000:80 hellodocker
```

Port **80** inside the container (where Flask listens) is mapped to port **4000** on the
host machine.

### 4. Test the application

```bash
curl http://localhost:4000
```

Expected response (the hostname is the container ID):

```html
<h3>Hello World!</h3><b>Hostname:</b> <container-id><br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```

The Redis visit counter cannot connect because only the Redis *library* is installed in
the image — no Redis *service* is running yet (this is covered in a later lab). The app
catches the `RedisError` and displays the "counter disabled" message instead. The app can
also be opened in a browser at <http://localhost:4000>.

### 5. Stop the container

Press `Ctrl + C` in the terminal where the container is running, or from another terminal:

```bash
docker container ls
docker container stop CONTAINER_ID
```

### 6. Push the image to Docker Hub

Create an account at <https://hub.docker.com/signup>, then:

```bash
docker login
docker tag hellodocker <your-dockerhub-username>/hellodocker:latest
docker push <your-dockerhub-username>/hellodocker:latest
```

---

## Part II – Deploy nginx on Kubernetes (Minikube)

Fork <https://github.com/pritamworld/DevOps_Kubernetes_exercise9> on GitHub and clone
your fork, or work directly with the `kubernetes/` folder of this repository (same files,
originally in the `k8/` folder of the instructor's repo).

Start Minikube first, then run the following commands from the folder that contains the
YAML files:

```bash
minikube start

kubectl apply -f nginx-deploy.yaml
kubectl apply -f nginx-service.yaml

kubectl rollout status deployment/nginx
kubectl get pods -l app=nginx -o wide
kubectl get svc nginx-nodeport
```

Access the nginx welcome page:

```bash
minikube service nginx-nodeport --url
# or, using the NodePort directly:
curl http://$(minikube ip):30080
```

Cleanup:

```bash
kubectl delete -f nginx-deploy.yaml
kubectl delete -f nginx-service.yaml
```

---

## Submission

- This GitHub repository (all code).
- A single document (DOCX/PDF) with a description, the executed command, and a terminal
  screenshot for every step of Part I and Part II.
- GitHub repository link submitted on D2L.
