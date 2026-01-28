# 📦 Spring Boot API (Dockerized)

This repository contains a Spring Boot REST API and a MySQL database environment. The project is fully containerized for easy deployment.

## 🛠️ Quick Start
delete all old docker volumes, images
```bash
# 1. Stop and remove ALL containers, networks, and images from previous labs
sudo docker stop $(sudo docker ps -aq)
sudo docker rm $(sudo docker ps -aq)

# 2. Prune unused networks and volumes (This is the 'Secret Sauce')
# This clears out old WordPress/MySQL volumes that squat on port 3306
sudo docker volume prune -f
sudo docker network prune -f

# 3. Double-check that the host MySQL is dead
sudo systemctl stop mysql
```

### 1. Clone the Project

Create a directory and pull the latest code:

```bash
mkdir -p ~/spring-projects && cd ~/spring-projects
git clone https://github.com/RithyHang/SpringAPI.git

```
```bash
cd SpringAPI/
```

### 2. Clean Port Conflicts

Before starting, ensure ports **8080** and **3306** are not being used by local services (like a local MySQL installation or a previous lab):

```bash
# Stop local MySQL if it's running
sudo systemctl stop mysql

# Force kill any process on the required ports
sudo fuser -k 8080/tcp
sudo fuser -k 3306/tcp

```

### 3. Build and Run

Use Docker Compose to build the Java application and start the services:

```bash
sudo docker-compose up --build -d

```

---

## 🚀 Accessing the API

Once the containers are started, you can verify they are running with `sudo docker ps`.

| Service | URL |
| --- | --- |
| **Swagger UI** | `http://<VM_IP_ADDRESS>:8080/swagger-ui/index.html` |
| **phpMyAdmin** | `http://<VM_IP_ADDRESS>:8081` |

> **Note:** To find your `<VM_IP_ADDRESS>`, run `ip addr` in your terminal and look for the `inet` address under your main network interface (usually `ens33` or `eth0`).

---

## ⚠️ Troubleshooting

* **Database Reset:** If you change environment variables in `docker-compose.yml` and the changes aren't reflecting, run `sudo docker-compose down -v` to wipe the persistent volumes and start fresh.
* **Logs:** If the API container is restarting, check the error logs:
```bash
sudo docker logs product_api_container

```
