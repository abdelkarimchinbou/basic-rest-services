# Basic REST Services - Microservices with Spring Boot 3

This project demonstrates a microservices architecture with Spring Boot 3 using Docker Compose. It includes:
- Product Service
- Recommendation Service
- Review Service
- Product Composite Service (aggregator)

## Quick Start

Build and run with Docker Compose:
```bash
./gradlew clean build
docker compose up --build -d
./test-em-all.bash start stop
```

---

## Essential Docker Commands

### Image Management
```bash
# List all images
docker image ls
# or
docker images

# Remove an image
docker image rm <image_id_or_name>

# Build an image
docker build -t <image_name>:<tag> <dockerfile_path>
```

### Container Management
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Run a container interactively
docker run -it <image_name>

# Run a container in background
docker run -d -p <host_port>:<container_port> <image_name>

# Stop a container
docker stop <container_id_or_name>

# Start a stopped container
docker start <container_id_or_name>

# Remove a container
docker rm <container_id_or_name>

# View container logs
docker logs <container_id_or_name>

# Follow container logs (real-time)
docker logs -f <container_id_or_name>

# Execute command in running container
docker exec -it <container_id_or_name> /bin/bash
```

### Container Inspection
```bash
# View container details
docker inspect <container_id_or_name>

# View container resource usage
docker stats

# View container IP address
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_id_or_name>
```

---

## Essential Docker Compose Commands

### Project Lifecycle
```bash
# Start all services defined in docker-compose.yml
docker compose up -d

# Start services and rebuild images
docker compose up --build -d

# Stop all running services (data preserved)
docker compose stop

# Start previously stopped services
docker compose start

# Stop and remove containers, networks (data in volumes preserved)
docker compose down

# Stop and remove everything including volumes
docker compose down -v

# Remove orphaned containers
docker compose down --remove-orphans
```

### Monitoring and Debugging
```bash
# View logs from all services
docker compose logs

# Follow logs from all services (real-time)
docker compose logs -f

# View logs from specific service
docker compose logs <service_name>

# Follow logs from specific service
docker compose logs -f <service_name>

# View last N lines of logs
docker compose logs --tail=50

# View service status
docker compose ps

# View service details
docker compose images
```

### Service Management
```bash
# Run command in service container
docker compose exec <service_name> <command>

# Example: Open shell in product-composite service
docker compose exec product-composite sh

# Rebuild services
docker compose build

# Rebuild specific service
docker compose build <service_name>

# Rebuild without cache
docker compose build --no-cache
```

### Network and Debugging
```bash
# Inspect docker compose network
docker network ls

# View network details
docker network inspect <network_name>

# Test connectivity between services (from within container)
docker compose exec product-composite curl http://recommendation:8080/recommendation?productId=1
```

---

## Project-Specific Commands

### Build and Test
```bash
# Clean build all Gradle projects
./gradlew clean build

# Build Docker images
docker compose build

# Run complete test suite (start, run tests, stop)
./test-em-all.bash start stop

# Run tests without stopping containers
./test-em-all.bash

# Stop the test environment
./test-em-all.bash stop
```

### Service URLs (when running with Docker Compose)
```bash
# Product Composite Service (publicly exposed)
http://localhost:8080/product-composite/1

# Direct service calls (from host, requires exposing ports in docker-compose.yml)
# Product Service: http://localhost:8081/product/1
# Recommendation Service: http://localhost:8082/recommendation?productId=1
# Review Service: http://localhost:8083/review?productId=1
```

### Testing Specific Features
```bash
# Test product with recommendations and reviews (productId=1)
curl -s http://localhost:8080/product-composite/1 | jq .

# Test product with no recommendations (productId=113)
curl -s http://localhost:8080/product-composite/113 | jq .

# Test product with no reviews (productId=213)
curl -s http://localhost:8080/product-composite/213 | jq .

# Test non-existent product (productId=13)
curl -s http://localhost:8080/product-composite/13 | jq .

# Test invalid productId (negative)
curl -s http://localhost:8080/product-composite/-1 | jq .

# Test invalid productId format
curl -s http://localhost:8080/product-composite/invalidProductId | jq .
```

---

## Troubleshooting

### Service Connection Issues
```bash
# Check if services are running
docker compose ps

# View service logs for errors
docker compose logs product-composite
docker compose logs recommendation
docker compose logs review
docker compose logs product

# Restart services
docker compose restart

# Restart specific service
docker compose restart <service_name>
```

### Container Connectivity
```bash
# Check service DNS resolution from within container
docker compose exec product-composite nslookup recommendation

# Test HTTP connectivity
docker compose exec product-composite curl -v http://recommendation:8080/recommendation?productId=1

# Check service ports
docker compose exec product-composite netstat -tlnp
```

### Clean Restart
```bash
# Complete cleanup and restart
docker compose down --remove-orphans -v
./gradlew clean build
docker compose up --build -d
./test-em-all.bash start stop
```

---

## Environment Variables

Key environment variables (set in docker-compose.yml):
- `SPRING_PROFILES_ACTIVE=docker` - Activates Docker profile for service configuration
- Service communication within Docker: service_name:8080 (e.g., `http://recommendation:8080/`)

---

## Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| Services cannot communicate | Ensure all services have docker profile configuration with port 8080 |
| Port already in use | `docker compose down -v && docker compose up -d` |
| Containers keep restarting | Check logs: `docker compose logs <service_name>` |
| Tests fail with connection error | Wait for services to be ready, check: `docker compose ps` |
| Need to debug | Use `docker compose exec <service> sh` to enter container shell |
