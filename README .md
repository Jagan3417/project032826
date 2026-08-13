# Flask MySQL Application

A two-tier web application built using **Python Flask** and **MySQL**, containerized with **Docker** and **Docker Compose**, with a **Jenkins pipeline** for automated build and deployment.

## Project Overview

The application provides a simple web interface for submitting and viewing messages.

The Flask application communicates with a MySQL database to store and retrieve messages.

The project consists of two main application services:

- **Flask** — Web application
- **MySQL** — Database

Docker Compose manages both services and their communication.

Jenkins is used to automate the application build and deployment process.

---

## Architecture

```text
                     GitHub Repository
                            |
                            v
                       Jenkins
                            |
                            v
                    Docker Image Build
                            |
                            v
                     Docker Compose
                            |
              +-------------+-------------+
              |                           |
              v                           v
       Flask Application             MySQL Database
          Container                    Container
          Port 5000                    Port 3306
              |                           |
              +-------- Docker Network ---+
                                          |
                                          v
                                   MySQL Data Volume
```

---

## Technology Stack

| Technology | Usage |
|---|---|
| Python | Application development |
| Flask | Web framework |
| MySQL | Database |
| Flask-MySQLdb | Flask/MySQL connectivity |
| Docker | Containerization |
| Docker Compose | Multi-container application management |
| Jenkins | CI/CD automation |
| GitHub | Source code repository |

---

## Repository Structure

```text
project032826/
│
├── app.py
├── Dockerfile
├── docker-compose.yml
├── Jenkinsfile
├── message.sql
├── requirements.txt
└── README.md
```

### `app.py`

Main Flask application.

It:

- Creates the Flask application.
- Configures the MySQL connection.
- Creates the `messages` table if it does not exist.
- Retrieves messages from MySQL.
- Inserts submitted messages into MySQL.
- Provides the application routes.

### `Dockerfile`

Defines the Docker image used to run the Flask application.

It:

- Uses Python 3.9 Slim.
- Sets the application working directory.
- Installs required system dependencies.
- Installs Python packages from `requirements.txt`.
- Copies the application files.
- Exposes port `5000`.
- Starts the Flask application.

### `docker-compose.yml`

Defines the application containers and their configuration.

It contains:

- Flask service
- MySQL service
- Docker network
- MySQL persistent volume
- Environment variables
- Port mappings
- Service health checks

### `Jenkinsfile`

Defines the Jenkins pipeline used to:

1. Checkout the source code.
2. Build the Docker image.
3. Deploy the application using Docker Compose.

### `message.sql`

Contains the SQL definition for the application's `messages` table.

### `requirements.txt`

Contains the Python packages required by the Flask application.

---

# Application

## Flask Application

The application is implemented in `app.py`.

The Flask application connects to MySQL using the following configuration:

```text
MYSQL_HOST
MYSQL_USER
MYSQL_PASSWORD
MYSQL_DB
```

The values are supplied through environment variables when the application runs with Docker Compose.

---

## Application Routes

### `GET /`

Retrieves messages from the MySQL database and displays them through the Flask template.

Application flow:

```text
Browser
   |
   | GET /
   v
Flask
   |
   | SELECT messages
   v
MySQL
   |
   | Return records
   v
Flask Template
   |
   v
Browser
```

### `POST /submit`

Accepts a message submitted by the user and stores it in the MySQL database.

Application flow:

```text
Browser
   |
   | POST /submit
   v
Flask
   |
   | INSERT message
   v
MySQL
   |
   | Commit
   v
Flask
   |
   v
Response
```

---

# Database

The application uses MySQL.

The database configured by Docker Compose is:

```text
devops
```

The application uses the following table:

```text
messages
```

## Messages Table

```sql
CREATE TABLE messages (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message TEXT
);
```

### Columns

| Column | Type | Description |
|---|---|---|
| `id` | INT | Primary key and auto-increment value |
| `message` | TEXT | Message submitted by the user |

The application also creates the table using `CREATE TABLE IF NOT EXISTS` when it starts.

---

# Docker

## Docker Image

The Flask application is packaged into a Docker image using the project's `Dockerfile`.

Build the image manually:

```bash
docker build -t flask-app:latest .
```

Run the image:

```bash
docker run -p 5000:5000 flask-app:latest
```

The application is exposed on:

```text
http://localhost:5000
```

---

# Docker Compose

Docker Compose runs the Flask application and MySQL database together.

## Services

### Flask

The Flask service:

- Builds from the project's `Dockerfile`.
- Runs the Flask application.
- Exposes port `5000`.
- Connects to the MySQL service.
- Uses the Docker Compose network.

The MySQL host is:

```text
mysql
```

The value is provided through:

```text
MYSQL_HOST=mysql
```

### MySQL

The MySQL service:

- Runs the MySQL database.
- Uses database `devops`.
- Exposes port `3306`.
- Stores database data in a Docker volume.
- Uses a health check.
- Runs on the same Docker network as Flask.

---

## Docker Network

The Flask and MySQL containers communicate through the Docker Compose network.

```text
Flask Container
      |
      | MySQL connection
      |
      v
MySQL Container
```

The Flask application connects to MySQL using the Compose service name:

```text
mysql
```

---

## Persistent Storage

MySQL data is stored using the Docker volume:

```text
mysql-data
```

The volume is mounted at:

```text
/var/lib/mysql
```

This allows MySQL data to persist when the MySQL container is recreated.

---

# Configuration

The Flask application uses the following MySQL environment variables:

```text
MYSQL_HOST=mysql
MYSQL_USER=root
MYSQL_PASSWORD=root
MYSQL_DB=devops
```

These values are configured by Docker Compose.

---

# Running the Application

## Prerequisites

The following tools are required:

- Git
- Docker
- Docker Compose

Verify Docker:

```bash
docker --version
```

Verify Docker Compose:

```bash
docker compose version
```

---

## Clone the Repository

```bash
git clone https://github.com/Jagan3417/project032826.git
```

Navigate to the project:

```bash
cd project032826
```

---

## Start the Application

Build and start both services:

```bash
docker compose up -d --build
```

Check the running containers:

```bash
docker compose ps
```

The application can then be accessed at:

```text
http://localhost:5000
```

---

## View Logs

View all service logs:

```bash
docker compose logs
```

Follow logs continuously:

```bash
docker compose logs -f
```

View Flask logs:

```bash
docker compose logs flask
```

View MySQL logs:

```bash
docker compose logs mysql
```

---

## Stop the Application

Stop and remove the containers:

```bash
docker compose down
```

To also remove the MySQL data volume:

```bash
docker compose down -v
```

---

# Jenkins Pipeline

The repository contains a `Jenkinsfile` defining the CI/CD pipeline.

The pipeline performs the following stages:

```text
Checkout
   |
   v
Docker Build
   |
   v
Docker Compose Deployment
```

## Checkout

Jenkins checks out the project source code from GitHub.

The pipeline uses the `main` branch.

## Docker Build

The Docker image is built using:

```bash
docker build -t flask-app:latest .
```

## Deployment

The existing Docker Compose deployment is stopped:

```bash
docker compose down || true
```

The application is then rebuilt and started:

```bash
docker compose up -d --build
```

---

# Complete Deployment Flow

```text
Developer
    |
    | Push code
    v
GitHub
    |
    | Jenkins checkout
    v
Jenkins
    |
    | Docker build
    v
Docker Image
    |
    | Docker Compose
    v
+-------------------------+
|                         |
|   Flask Container       |
|          |              |
|          |              |
|          v              |
|   MySQL Container       |
|                         |
+-------------------------+
           |
           v
    Persistent Volume
```

---

# Docker Commands

### List running containers

```bash
docker ps
```

### List all containers

```bash
docker ps -a
```

### List Docker images

```bash
docker images
```

### View Compose services

```bash
docker compose ps
```

### Rebuild application

```bash
docker compose up -d --build
```

### Restart services

```bash
docker compose restart
```

### Stop services

```bash
docker compose down
```

### Remove services and volumes

```bash
docker compose down -v
```

---

# Project Components

```text
GitHub
  |
  v
Jenkins
  |
  v
Docker
  |
  +------------------+
  |                  |
  v                  v
Flask              MySQL
  |                  |
  +------ Network ---+
                     |
                     v
              mysql-data volume
```

The project combines source control, CI/CD, containerization, application development, database integration, networking, and persistent storage into a single application deployment.