# Lab: Custom Build and Cloud Integration (Spring Boot + AWS RDS)

### 1. The Concept: "Build vs. Orchestrate"

In this lab, we use a two-step process to get the application running:

* **The Dockerfile (The Build):** Compiles the Java source code using Gradle and packages it into a lightweight container image.
* **Docker Compose (The Orchestration):** Manages the "Runtime." It maps the ports and injects the **AWS RDS** connection credentials into the container environment.

---

### 2. Project Folder Structure

Ensure your Ubuntu project folder looks like this:

```text
~/ProductAPI_Project/
├── Dockerfile              # The build instructions
├── docker-compose.yaml     # The runtime configuration
├── build.gradle            # Build dependencies
├── gradlew                 # Gradle wrapper
└── src/                    # Your Java source code

```

---

### 3. The Build Instructions (`Dockerfile`)

This file turns your code into an image. We use the **JDK 21** base image to match your Spring Boot version.

```dockerfile
# Use official Java 21 image
FROM eclipse-temurin:21-jdk

WORKDIR /app

# Copy project files
COPY . .

# Make gradlew executable and build the JAR
RUN chmod +x gradlew
RUN ./gradlew clean build -x test

# Expose port 8080 and run the JAR
EXPOSE 8080
CMD ["java", "-jar", "build/libs/ProductAPI-0.0.1-SNAPSHOT.jar"]

```

---

### 4. The Orchestration (`docker-compose.yaml`)

This file connects your container to the outside world and your AWS Database.

```yaml
version: '3.8'

services:
  product_api:
    build: .             # Build image using the Dockerfile in current folder
    container_name: product_api_container
    ports:
      - "8080:8080"      # Access via http://<VM_IP>:8080
    environment:
      # Injecting AWS RDS Credentials
      - SPRING_DATASOURCE_URL=jdbc:mysql://your-rds-endpoint.aws.com:3306/your_db
      - SPRING_DATASOURCE_USERNAME=admin
      - SPRING_DATASOURCE_PASSWORD=your_password
    restart: always

```

---

### 5. Running the Project

Since you are building from source, you must use the `--build` flag to ensure any code changes are captured.

```bash
# 1. Build and Start
sudo docker-compose up --build -d

# 2. Check the Logs (Verify RDS Connection)
sudo docker logs -f product_api_container

# 3. Test the Swagger UI
# Open in browser: http://yourIPaddressHere:8080/swagger-ui/index.html

```
