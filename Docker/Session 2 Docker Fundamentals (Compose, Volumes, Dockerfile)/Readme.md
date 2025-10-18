# Lab02 — Dockerizing a Flask Microservice & Local Development Environment

This repository contains two tasks from **Lab 02**.

---

## Task 1 — Dockerize Flask App

### Commands to Build & Run

```bash
# Build the image
docker build -t flask-app .

# Run the container
docker run -d -p 5000:5000 --name flask-container flask-app

# Verify it’s running
curl http://localhost:5000
```

---

## Task 2 — Docker Compose with MySQL & NGINX

### Commands to Run & Verify

```bash
# Start all services
docker compose up -d

# Check running containers
docker ps

# Test NGINX access from browser or curl
curl http://localhost:8090

# Verify MySQL is healthy
docker logs mysql-db

# Stop all containers
docker compose down
```

---

📁 **Repository Structure**

```
lab02-docker/
├─ task1-dockerize flask app/
│  ├─ app.py
│  ├─ requirement.txt
│  └─ Dockerfile
├─ task2- compose application/
│  └─ docker-compose.yml
└─ README.md
```
