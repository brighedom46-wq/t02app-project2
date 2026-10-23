# Project 2: Containerization and Local Serverless Function

## Overview

This project demonstrates containerization using Docker, multi-stage image builds, service orchestration using Docker Compose, and local serverless function execution using AWS SAM CLI.

## Technologies

- Docker and Docker Compose
- Node.js 24
- Nginx Alpine
- Redis 7 Alpine
- AWS SAM CLI
- AWS Lambda with Node.js 22

## Prerequisites

Install Docker, Docker Compose, and AWS SAM CLI. Ensure Docker is running.

All commands below should be executed in an Ubuntu (WSL) terminal.

## 1. Build Docker Images

From the project root:

```bash
docker build -t demo:multi .
docker build -f Dockerfile.single -t demo:single .
docker images | grep demo
```

Observed image sizes:

- Multi-stage image: 93.6 MB
- Single-stage image: 1.65 GB

The multi-stage build produces a smaller runtime image by separating the Node.js build environment from the final Nginx image.

## 2. Run the Web Container

```bash
docker run --rm -d --name web01 -p 8080:80 demo:multi
curl http://localhost:8080
docker stop web01
```

Expected response:

```html
<!doctype html><html><body><h1>T02 demo</h1></body></html>
```

## 3. Run Docker Compose

```bash
docker compose up -d
docker compose ps
```

The application contains two services:

- `web`: Nginx serving the static web application on port 8080.
- `cache`: Redis with a health check.

Test both services:

```bash
curl http://localhost:8080
docker compose exec cache redis-cli ping
```

Expected Redis response: `PONG`.

Stop the services:

```bash
docker compose down
```

## 4. Invoke the Local Lambda Function

```bash
cd local-fn
sam local invoke HelloWorldFunction
sam local invoke HelloWorldFunction -e events/event.json
cd ..
```

Expected function response includes:

```json
{"statusCode":200,"body":"{\"message\":\"hello world\"}"}
```

AWS credentials are not required for local invocation.

## 5. Debugging and Inspection

View Nginx logs:

```bash
docker compose logs web
```

Check Nginx and Redis:

```bash
docker compose exec web sh -c 'nginx -v'
docker compose exec cache redis-cli ping
```

Inspect container status:

```bash
docker inspect $(docker compose ps -q web) --format '{{.State.Status}} started={{.State.StartedAt}} restarts={{.RestartCount}}'
```

## 6. Dependency Failure Test

```bash
docker compose stop cache
docker compose ps
curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8080
docker compose down
```

The web service continues returning HTTP 200 because Redis is a startup dependency in this demonstration, not a runtime dependency used by the static web application.

## Conclusion

This project demonstrates how multi-stage Docker builds reduce image size, Docker Compose manages multiple services, and AWS SAM enables local testing of serverless functions without deploying to AWS.
