## **Docker Task 1: Multi-Network Application Deployment**

**Overview**  
This project demonstrates how to deploy a multi-container application using Docker, with a clear separation between the database backend and the application frontend using custom Docker networks. The setup also includes a load balancer (HAProxy) to distribute traffic between multiple application instances.

---

### **Architecture**

- **Database Layer:**  
  - A MySQL container runs in its own isolated network (`petclinic-db-net`), storing application data in a persistent Docker volume.
- **Application Layer:**  
  - Two instances of the application run in containers, initially connected to the database network for backend access.
  - One instance is also connected to a separate frontend network (`petclinic-app-net`) to allow access from the load balancer.
- **Load Balancer:**  
  - An HAProxy container runs in the frontend network, forwarding external traffic to the application containers.

---

### **Project Structure**
```
├── Dockerfile.mvn 
├── Dockerfile.grd
├── .dockerignore
├── README.md
├── haproxy.cfg
└── [application source code]
```

---

### **How to Build the Application Image**

This project uses a **multi-stage Dockerfile** approach (see `Dockerfile.mvn` and `Dockerfile.grd`) to efficiently build and package the application:

- **Builder Stage:**  
  The first stage uses a full JDK image with Maven or Gradle to compile the source code, run tests, and build the application artifact (such as a JAR file).
- **Runtime Stage:**  
  The second stage uses a lightweight JRE image to run the application. Only the built artifact and necessary runtime files are copied from the builder stage, resulting in a much smaller and more secure final image.

**For Maven projects:**
```sh
docker build -f Dockerfile.mvn -t petclinic-app:v1.1 .
```

**For Gradle projects:**
```sh
docker build -f Dockerfile.grd -t petclinic-app:v1.2 .
```

### **How to Run**

1. **Create a persistent volume for the database:**
   ```sh
   docker volume create petclinic-db
   ```

2. **Create custom Docker networks:**
   ```sh
   docker network create --driver bridge --internal --subnet 10.0.1.0/24 petclinic-db-net
   docker network create --driver bridge --subnet 10.0.2.0/24 petclinic-app-net
   ```

3. **Start the MySQL database container:**
   ```sh
   docker run -d \
     -e MYSQL_USER=petclinic \
     -e MYSQL_PASSWORD=petclinic \
     -e MYSQL_ROOT_PASSWORD=root \
     -e MYSQL_DATABASE=petclinic \
     --name mysql-db \
     --network petclinic-db-net \
     -v petclinic-db:/var/lib/mysql \
     mysql:9.2
   ```

4. **Start the first application instance:**
   ```sh
   docker run -d \
     --name spring-app \
     --network petclinic-db-net \
     -p 8083:8080 \
     -e MYSQL_URL=jdbc:mysql://mysql-db:3306/petclinic \
     -e SPRING_PROFILES_ACTIVE=mysql \
     -e MYSQL_USER=petclinic \
     -e MYSQL_PASS=petclinic \
     petclinic-app:v1.2
   ```
   Access at: [http://localhost:8083](http://localhost:8083)

5. **Start the second application instance:**
   ```sh
   docker run -d \
     --name spring-app-2 \
     --network petclinic-db-net \
     -p 8084:8080 \
     -e MYSQL_URL=jdbc:mysql://mysql-db:3306/petclinic \
     -e SPRING_PROFILES_ACTIVE=mysql \
     -e MYSQL_USER=petclinic \
     -e MYSQL_PASS=petclinic \
     petclinic-app:v1.2
   ```

6. **Connect the second app instance to the frontend network:**
   ```sh
   docker network connect petclinic-app-net spring-app-2
   ```
   Access at: [http://localhost:8084](http://localhost:8084)

7. **Start the HAProxy load balancer:**
   ```sh
   docker run -d \
     --name spring-lb \
     --network petclinic-app-net \
     -p 8080:80 \
     -v $(pwd)/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro \
     haproxy:alpine
   ```
   Access the load-balanced application at: [http://localhost:8080](http://localhost:8080)

---

### **Key Concepts**

- **Network Isolation:**  
  The database and application layers are separated by Docker networks for security and modularity.
- **Persistence:**  
  Database data is stored in a Docker volume to survive container restarts.
- **Scalability:**  
  Multiple application instances can be added and balanced using HAProxy.
- **Configuration:**  
  The `haproxy.cfg` file defines how traffic is distributed between application containers.

---