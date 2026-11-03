# Docker command reference

Run commands from the project root.

## Build and start
```bash
docker compose config
docker compose build
docker compose up -d
docker compose ps
```

## Logs and inspection
```bash
docker compose logs -f
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f database
docker ps -a
docker inspect student-backend
```

## Container shells
```bash
docker compose exec backend sh
docker compose exec database sh
```

## Restart and stop
```bash
docker compose restart backend
docker compose down
docker compose up -d
```

## Rebuild after source changes
```bash
docker compose build backend frontend
docker compose up -d
```

## Database volume
```bash
docker volume ls
docker volume inspect student-registration-docker_student_db_data
```

Warning: `docker compose down -v` deletes the database volume and its data.
