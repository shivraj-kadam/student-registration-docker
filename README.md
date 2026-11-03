# Student Registration Application — Docker Deployment

A Docker-based deployment structure for a three-service student registration application:

- **Frontend:** React + Vite, served by Nginx
- **Backend:** Spring Boot application (Java 17 / Maven)
- **Database:** MariaDB 11 with a persistent Docker volume

> **Important:** This repository currently contains Docker deployment/configuration files. Add your original application source before building: backend Maven project (including `pom.xml` and `src/`) and frontend Vite project (including `package.json`, lock file and `src/`). These source files cannot be reconstructed from terminal commands alone.

## 1. Architecture

```text
Browser
  |
  | HTTP :80
  v
Frontend container (Nginx)
  |
  | /api/* reverse proxy
  v
Backend container (Spring Boot :8080)
  |
  | JDBC / internal Docker network
  v
MariaDB container (:3306)
  |
  v
Named volume: student_db_data
```

The frontend and backend communicate through Docker Compose service names. The database is not published to the host by default.

## 2. Repository structure

```text
student-registration-docker/
├── README.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── DOCKER_COMMANDS.md
├── backend/
│   ├── Dockerfile
│   └── .dockerignore
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   └── .dockerignore
└── database/
    └── Dockerfile
```

Add your actual backend and frontend source directories to the corresponding folders.

## 3. Prerequisites

Install these on your Ubuntu EC2 instance or other Docker host:

- Docker Engine
- Docker Compose plugin
- Git

Verify installations:

```bash
docker --version
docker compose version
git --version
```

## 4. Get the project

```bash
git clone https://github.com/shivraj-kadam/student-registration-docker.git
cd student-registration-docker
```

Copy your original application source into the expected folders:

- Backend: `backend/pom.xml` and `backend/src/`
- Frontend: `frontend/package.json`, lock file and `frontend/src/`

Do not overwrite the Dockerfiles unless you have checked that the application build paths still match.

## 5. Configure environment variables

Create a local environment file:

```bash
cp .env.example .env
nano .env
```

Set strong, unique database credentials and confirm the backend database name/user/password match the Compose configuration. The `.env` file is local and must never be committed.

## 6. Build and start all containers

From the directory containing `docker-compose.yml`:

```bash
docker compose config
docker compose build
docker compose up -d
```

Check running containers:

```bash
docker compose ps
docker ps
```

View logs:

```bash
docker compose logs -f
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f database
```

## 7. Test the application

Open the Docker host's public DNS name or IP in a browser:

```text
http://YOUR_EC2_PUBLIC_IP/
```

The Nginx configuration forwards requests beginning with `/api/` to the backend service on the internal Docker network. Test your API endpoint through the frontend host, for example:

```text
http://YOUR_EC2_PUBLIC_IP/api/users
```

The exact API routes depend on your Spring Boot application.

## 8. Useful container operations

```bash
# Stop containers (keeps database data)
docker compose down

# Start again
docker compose up -d

# Restart one service
docker compose restart backend

# Follow logs
docker compose logs -f backend

# List containers
docker ps -a

# Inspect a container
docker inspect student-backend

# Open a shell in the backend container
docker compose exec backend sh
```

To remove containers and the database volume permanently:

```bash
docker compose down -v
```

**Warning:** `-v` deletes the MariaDB data stored in the named volume. Do not run it unless you intentionally want to delete the database.

## 9. How the Docker network works

Compose creates an internal network for the services. The backend connects to MariaDB using the service hostname `database`, not `localhost` and not the EC2 public IP. The frontend Nginx container connects to the backend using `backend:8080`.

Only the frontend port is published by default. Keep database port 3306 private. Do not expose MariaDB publicly.

## 10. Updating the application

After changing backend or frontend source code:

```bash
docker compose build backend frontend
docker compose up -d
docker compose ps
```

Check logs if a service does not become healthy:

```bash
docker compose logs --tail=100 backend
docker compose logs --tail=100 frontend
docker compose logs --tail=100 database
```

## 11. Push future changes to GitHub

```bash
git status
git add .
git commit -m "Update Docker student registration application"
git push origin main
```

Never commit `.env`, private SSH keys, AWS credentials, database dumps containing real user data, or other secrets.

## 12. Troubleshooting

| Problem | What to check |
|---|---|
| Frontend does not load | EC2 security group allows inbound TCP 80; frontend container and Nginx logs |
| API returns 502 | Backend container is running; Nginx upstream name is `backend`; backend listens on port 8080 |
| Backend cannot connect to DB | JDBC host is `database`; credentials and database name match; database logs |
| Container exits | Run `docker compose logs SERVICE_NAME` |
| Changes are not visible | Rebuild the changed image and recreate the service |

## 13. Security checklist

- Use strong database credentials in the local `.env` file.
- Keep `.env` out of Git.
- Do not expose database port 3306 to the internet.
- Restrict EC2 security group inbound rules to required ports.
- Use HTTPS for a public production deployment.
- Back up the database volume before upgrades or destructive operations.

## 14. Project learning outcomes

This project demonstrates Docker images and containers, Dockerfiles, Docker Compose, container networking, environment variables, persistent volumes, Nginx reverse proxying, application logs, and multi-container deployment.

---

**Maintainer:** shivraj-kadam  
**Repository:** https://github.com/shivraj-kadam/student-registration-docker
