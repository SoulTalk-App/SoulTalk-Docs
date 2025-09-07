# SoulTalk Infrastructure - Local Development

This directory contains the minimal infrastructure needed to run SoulTalk locally for development.

## What's Included

- **Docker Compose**: All services running locally
- **Development Scripts**: Helper scripts for common tasks
- **Local Configuration**: Environment setup for development

## Services

- **Keycloak**: Authentication server (http://localhost:8080)
- **PostgreSQL**: Database for Keycloak (port 5433)
- **Redis**: Caching and sessions (port 6379)
- **Backend API**: FastAPI server (http://localhost:8000)
- **Mailhog**: Email testing (http://localhost:8025)

## Quick Start

1. **Start all services**:
   ```bash
   ./scripts/start.sh
   ```

2. **Stop all services**:
   ```bash
   ./scripts/stop.sh
   ```

3. **View logs**:
   ```bash
   ./scripts/logs.sh [service-name]
   ```

4. **Reset everything**:
   ```bash
   ./scripts/reset.sh
   ```

## Development Workflow

1. **Start services** → `./scripts/start.sh`
2. **Start mobile app** → `cd ../SoulTalk-Mobile && npm start`
3. **Develop** → Make changes to backend or mobile app
4. **Test** → Use the mobile app or API directly
5. **Check emails** → Visit http://localhost:8025

That's it! No complex infrastructure, just simple local development.