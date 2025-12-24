# Microservices Orchestrator

This repository is a **microservices orchestrator**.  
It keeps the shared infrastructure (Docker Compose, documentation) and links each microservice as a **Git submodule**.

## Services

- **API-Gateway** (`./API-Gateway`) — entry point / routing to backend services
- **Authentication-Service** (`./Authentication-Service`) — authentication and JWT issuing/validation
- **User-Service** (`./User-Service`) — user management (DB + Redis)
- **Order-Service** (`./Order-Service`) — order management
- **Payment-Service** (`./Payment-Service`) — payments (optional / if enabled in compose)

## Documentation

- The usage guide for the orchestrator is located in: `./docs/`
    - Example: `docs/Microservices-Orchestrator-Guide.md`

## Environment variables (.env)

Each service folder may contain its own `.env` (or `.env.example`) for **service-local** development.

The **orchestrator** is started with `docker-compose.yml`, and it reads variables from:
- your shell environment, and/or
- an optional root `.env` file (if you create it next to `docker-compose.yml`)

> Note: In the current compose setup, environment variables are defined inline with defaults like `${VAR:-default}`.
> If you want Compose to load variables automatically, create a root `.env` file in the orchestrator directory.

## OpenAPI / Swagger

Each microservice provides an **OpenAPI (Swagger) specification** for its endpoints.

Typical locations:
- Swagger UI: `http://localhost:<service-port>/swagger-ui.html`
- OpenAPI JSON: `http://localhost:<service-port>/v3/api-docs`

(Exact paths/ports depend on the service configuration.)

## Quick start (Docker Compose)

From the repository root:

```bash
# Build and start everything
docker compose up -d --build

# Follow logs
docker compose logs -f

# Stop and remove containers
docker compose down
```

### Ports (default)

- Authentication-Service: `8082`
- User-Service: `8081` (mapped to container `8080`)
- Order-Service: `8083`
- API-Gateway: `8084`

## Update submodules

```bash
git pull
git submodule update --init --recursive
```
