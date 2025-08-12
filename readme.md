Spring PetClinic with MySQL (Docker Compose)
This repository provides a Docker Compose setup for running the Spring PetClinic application with a MySQL database.
It uses separate containers for the application and database, connected through a custom Docker bridge network.

Features
Spring Boot PetClinic application (joshua122/petclinic:dev)

MySQL 5.7 database with preconfigured credentials

Isolated Docker network for secure communication

Simple orchestration via Docker Compose

Multi-stage Docker build for efficient packaging

Prerequisites
Before you begin, ensure you have installed:

Docker (latest stable)

Docker Compose

Optional (for local builds without Docker):

Java 17+

Maven

Running PetClinic with Docker Compose
1️⃣ Clone the repository
bash
Copy code
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
2️⃣ Build & Run
bash
Copy code
docker-compose up --build -d
This will:

Build the PetClinic application image from the Dockerfile

Start a MySQL 5.7 container with the required environment variables

Start the PetClinic application container

3️⃣ Verify running containers
bash
Copy code
docker ps
You should see:

app → running on port 8080

db → running on port 3306

4️⃣ Access the application
http://localhost:8080

5️⃣ Stop the services
bash
Copy code
docker-compose down

Docker Compose Configuration
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
      
Multi-Stage Dockerfile
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

Project Structure
css
Copy code


├── Dockerfile
├── docker-compose.yml
├── mvnw
├── mvnw.cmd
├── pom.xml
├── push-to-pws/
├── README.md
└── src/

Network Diagram
yaml

Copy code
┌────────────┐       ┌───────────┐
│   app      │◄─────►│    db     │
│ 8080       │       │  3306     │
└────────────┘       └───────────┘
       │
  localhost
Troubleshooting
Network not found: Ensure you run docker-compose up from the directory containing docker-compose.yml.

Port conflicts: Ensure ports 8080 (app) and 3306 (MySQL) are free.

Database connection issues: Check MySQL service health and confirm environment variables match the Spring configuration.

License
This project is licensed under the Apache License 2.0.
