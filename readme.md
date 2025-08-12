Spring PetClinic with MySQL (Docker Compose)
This repository provides a ready-to-run Spring PetClinic application integrated with a MySQL database using Docker Compose.
It leverages separate containers for the application and database, connected via a custom Docker bridge network for secure communication.

📦 Features
Spring Boot PetClinic application (joshua122/petclinic:dev)

MySQL 5.7 database with preconfigured credentials

Custom Docker bridge network for container isolation

Multi-stage Docker build for efficient packaging

Simple orchestration via Docker Compose

🛠 Prerequisites
Before starting, ensure you have installed:

Docker (latest stable version)

Docker Compose

(Optional) For local builds without Docker:

Java 17+

Maven

🚀 Getting Started
1️⃣ Clone the repository
bash
Copy code
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
2️⃣ Build & Run with Docker Compose
bash
Copy code
docker-compose up --build -d
This will:

Build the PetClinic application image from the Dockerfile.

Start a MySQL 5.7 container with configured credentials.

Start the Spring PetClinic container.

3️⃣ Verify running containers
bash
Copy code
docker ps
You should see:

app → running on port 8080

db → running on port 3306

4️⃣ Access the application
Open in your browser:

arduino
Copy code
http://localhost:8080
5️⃣ Stop the services
bash
Copy code
docker-compose down
⚙ Configuration
Service	Port	Image	Network
app	8080	joshua122/petclinic:dev	petclinic
db	3306	mysql:5.7	petclinic

Database credentials (from docker-compose.yml):

ini
Copy code
MYSQL_USER=petclinic
MYSQL_PASSWORD=petclinic
MYSQL_DATABASE=petclinic
MYSQL_ALLOW_EMPTY_PASSWORD=true
📄 docker-compose.yml
yaml
Copy code
version: "3.8"

networks:
  petclinic:
    driver: bridge

services:
  db:
    image: mysql:5.7
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD:
      MYSQL_ALLOW_EMPTY_PASSWORD: "true"
      MYSQL_USER: petclinic
      MYSQL_PASSWORD: petclinic
      MYSQL_DATABASE: petclinic
    networks:
      - petclinic

  app:
    image: joshua122/petclinic:dev
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: mysql
    networks:
      - petclinic
    depends_on:
      - db
🗂 Project Structure
css
Copy code
.
├── Dockerfile
├── docker-compose.yml
├── mvnw
├── mvnw.cmd
├── pom.xml
├── push-to-pws/
├── readme.md
└── src/
🖥 Docker Network Diagram
yaml
Copy code
┌────────────┐       ┌───────────┐
│   app      │◄─────►│    db     │
│ 8080       │       │  3306     │
└────────────┘       └───────────┘
       │
  localhost
🖥 Dockerfile (Multi-stage Build)
dockerfile
Copy code
FROM schoolofdevops/maven:spring AS build
WORKDIR /app
COPY . .
RUN mvn package -DskipTests

FROM build AS test
CMD mvn clean test

FROM eclipse-temurin:17-jre-alpine AS run
WORKDIR /app
COPY --from=build /app/target/spring-petclinic-2.3.1.BUILD-SNAPSHOT.jar petclinic.jar
EXPOSE 8080
CMD ["java", "-jar", "petclinic.jar"]
🔍 Troubleshooting
Network not found → Ensure you run docker-compose up from the folder containing docker-compose.yml.

Port conflicts → Ensure ports 8080 (app) and 3306 (MySQL) are free before starting.

Database connection issues → Check MySQL container health and confirm environment variables match Spring config.
