# Microservices Containerization Assessment

## Overview

This project containerizes four Node.js/Express microservices and runs them together with Docker Compose:

| Service | Container port | Local URL |
|---|---:|---|
| User Service | 3000 | http://localhost:3000 |
| Product Service | 3001 | http://localhost:3001 |
| Order Service | 3002 | http://localhost:3002 |
| Gateway Service | 3003 | http://localhost:3003 |

The assignment overview names User, Product, and Gateway services, while the provided repository also contains an Order Service and the Gateway routes requests to it. This submission includes all four services to preserve the supplied application's functionality.

## Prerequisites

- Docker Desktop installed and running (Windows/macOS/Linux)
- Docker Compose v2 (`docker compose`)
- Git (only needed for the GitHub steps)

## Folder structure

```text
submission/
├── user-service/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
├── product-service/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
├── order-service/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
├── gateway-service/
│   ├── app.js
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
└── README.md
```

## Start the application

Open a terminal in the folder containing `docker-compose.yml`.

Build the images and start all services in the background:

```bash
docker compose up --build -d
```

Check status and health:

```bash
docker compose ps
```

View combined logs:

```bash
docker compose logs -f
```

View one service's logs, for example:

```bash
docker compose logs -f gateway-service
```

## Test each service

Open these URLs in a browser, or run the commands below in PowerShell.

### User Service

- Health: http://localhost:3000/health
- Users: http://localhost:3000/users

```powershell
Invoke-RestMethod http://localhost:3000/health
Invoke-RestMethod http://localhost:3000/users
```

### Product Service

- Health: http://localhost:3001/health
- Products: http://localhost:3001/products

```powershell
Invoke-RestMethod http://localhost:3001/health
Invoke-RestMethod http://localhost:3001/products
```

### Order Service

- Health: http://localhost:3002/health
- Orders: http://localhost:3002/orders

```powershell
Invoke-RestMethod http://localhost:3002/health
Invoke-RestMethod http://localhost:3002/orders
```

Create a sample order through the Order Service:

```powershell
$body = @{ userId = 1; productId = 1 } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://localhost:3002/orders -ContentType "application/json" -Body $body
```

Orders are stored in memory, so they reset when the container restarts.

### Gateway Service

- Health: http://localhost:3003/health
- Users via gateway: http://localhost:3003/api/users
- Products via gateway: http://localhost:3003/api/products
- Orders via gateway: http://localhost:3003/api/orders

```powershell
Invoke-RestMethod http://localhost:3003/health
Invoke-RestMethod http://localhost:3003/api/users
Invoke-RestMethod http://localhost:3003/api/products
Invoke-RestMethod http://localhost:3003/api/orders
```

Create an order through the gateway:

```powershell
$body = @{ userId = 1; productId = 2 } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://localhost:3003/api/orders -ContentType "application/json" -Body $body
```

## Network and startup order

All services join the Compose-managed `microservices-net` bridge network. The gateway calls the other containers using their Compose service names (`user-service`, `product-service`, and `order-service`), not `localhost`. Health checks are configured, and Compose waits for the three backend services to become healthy before starting the gateway.

## Stop the application

Stop and remove the containers and network:

bash
docker compose down

Rebuild after changing application code or a Dockerfile:

bash
docker compose up --build -d


## Troubleshooting

1. **Docker daemon not running:** Start Docker Desktop, wait until it reports that Docker is running, and retry.
2. **Port already in use:** Stop the process using ports 3000–3003, or change the host-side port in `docker-compose.yml` (the number before the colon).
3. **A service exits:** Run `docker compose ps -a` and `docker compose logs <service-name>`.
4. **Gateway returns a 500 error:** Check `docker compose ps` and the backend logs. Confirm all services are attached to `microservices-net`.
5. **Dependencies fail to install:** Check internet connectivity and retry `docker compose build --no-cache`.
6. **Changes are not visible:** Run `docker compose up --build -d` to rebuild and recreate containers.
7. **Health shows starting/unhealthy:** Inspect that service's logs and verify its `/health` endpoint.

## Screenshots required for assessment

After running the project locally, capture genuine screenshots and add them to a `screenshots/` folder (or insert them into this README) before final submission. Recommended evidence:

1. Docker Desktop showing all four containers running.
2. Terminal output of `docker compose ps` showing services healthy.
3. Browser or terminal output for `/users`, `/products`, `/orders`, and the gateway routes.
4. Terminal showing a successful `docker compose up --build -d`.

**Do not use fabricated screenshots.** The ZIP is prepared with the screenshot checklist, but screenshots must be captured on the machine where Docker is actually run.

## GitHub submission

1. Fork the provided repository: https://github.com/dhikalerahulrd-a11y/Skill-Test-1.git
2. Copy the contents of this submission folder into your fork's repository root (or preserve the `submission/` folder if your assessor specifically requires it).
3. Commit and push the files.
4. Submit your fork URL and this ZIP file.

Example commands, after copying these files into your cloned fork:

bash
git add .
git commit -m "Add Dockerfiles and Docker Compose configuration"
git push origin main


## Assessment note

The Dockerfiles use the official Node.js 20 Alpine image, set a working directory, install production dependencies, expose each service's application port, and start the service with `node app.js`. The Compose file publishes each port and defines a shared network.
