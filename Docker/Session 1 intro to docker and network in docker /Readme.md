
# Lab 01 – Docker Network Configuration and Service Isolation

This repository contains my solution for **Lab 01: Docker Network Configuration and Service Isolation**, part of the **DevOps Engineer Diploma** at **Senior Academy**.

---

## Lab Objectives

- Custom Bridge Network Creation  
- IP Address Allocation and Management  
- Service Discovery and Internal Connectivity  
- Multi-Network Container Setup  
- Network Isolation and Security Validation  

---

## Task 1 – Custom Bridge Network for HR Application

### Requirements
- **Network Type:** Custom Bridge  
- **Network Name:** `hr-app-net`  
- **Subnet:** `192.168.20.0/24`  
- **Gateway:** `192.168.20.1`  
- **Containers:**
  - `nginx-server` → acts as the web frontend
  - `alpine-tester` → used for internal diagnostics and connectivity tests

### Steps
1. **Create the network**
   ```bash
   docker network create --driver bridge  --subnet 192.168.20.0/24  --gateway 192.168.20.1 hr-app-net
   ```

2. **Inspect the network**

   ```bash
   docker network inspect hr-app-net
   ```

   ![inspection for hr-app-net](../images/lab1/inspection%20for%20hr-app-net.png)


3. **Run containers**

   ```bash
   $ docker run -d -it  --name  alpine-tester --memory 100m  --cpus "0.5"  --network hr-app-net --entrypoint sh  alpine:latest

   $ docker run -d  --name  nginx-server --memory 100m  --cpus "0.5"  --network hr-app-net --entrypoint "sh"  nginx:alpine
   ```

4. **Verify IP allocation**

   ```bash
   docker inspect nginx-server | grep IPAddress
   ```
   ![inspection nginx-server](../images/lab1/inspection%20nginx-server.png)
   
   
   ```bash
   docker inspect alpine-tester | grep IPAddress
   ```
   ![inspection alpine-tester](../images/lab1/inspection%20alpine-tester.png)

5. **Test connectivity**

   ```bash
   docker exec nginx-server ping 192.168.20.3
   docker exec alpine-tester ping 192.168.20.2
   ```
   ![Pinging Pong](../images/lab1/Ping%20pong%20task1%20.png)

--- 

## Task 2 – Multi-Network Container Architecture

### Requirements

* **Network 1 (Frontend):**

  * Name: `frontend-net`
  * Subnet: `10.1.1.0/24`
* **Network 2 (Backend):**

  * Name: `backend-net`
  * Subnet: `10.1.2.0/24`
* **Containers:**

  * `nginx-lb` → connected to both networks (multi-homed)
  * `client-tester` → connected only to `frontend-net`
  * `backend-db` → connected only to `backend-net`

###  Steps

1. **Create both networks**

   ```bash
   docker network create --subnet 10.1.1.0/24 frontend-net
   docker network create --subnet 10.1.2.0/24 --internal backend-net
   ```
   ![inspection frontend and backend](../images/lab1/inspection%20frontend%20and%20backend.png)

2. **Deploy containers**

   ```bash
    $ docker container run -d -it   --name  client-tester  --network  frontend-net  alpine:latest
    $ docker container run -d -it   --name   backend-db  --network  backend-net  alpine:latest
    $ docker container run -d --name nginx-lb --network backend-net --network  frontend-net  nginx:alpine
   ```
3. **Verify IPs for Nginx**

   ```bash
   docker inspect nginx-lb | grep IPAddress
   ```
   ![inspection Nginx](../images/lab1/inspection%20Nginx.png)

4. **Test isolation**

   ```bash
   docker exec -it client-tester ping backend-db  # should fail
   docker exec -it nginx-lb ping backend-db       # should succeed
   ```
   ![Ping Pong ](../images/lab1/Ping%20pong%20task%202%20.png)


