# SoulTalk Development Guide

Simple, minimal setup for local development.

## Prerequisites

- Docker Desktop (running)
- Node.js 18+ 
- Python 3.11+ (optional, for backend development)

## Quick Start (2 minutes)

1. **Start everything**:
   ```bash
   ./SoulTalk-Infra/scripts/start.sh
   ```

2. **Start mobile app**:
   ```bash
   cd SoulTalk-Mobile
   npm install
   npm start
   ```

3. **Open mobile app**: http://localhost:8081

That's it! You're ready to develop.

## What's Running

| Service | URL | Purpose |
|---------|-----|---------|
| Backend API | http://localhost:8000 | Authentication, API endpoints |
| Keycloak | http://localhost:8080 | User authentication server |
| Mailhog | http://localhost:8025 | Email testing (catch all emails) |
| Mobile App | http://localhost:8081 | React Native app in browser |

## Development Workflow

### Testing Authentication Flow

1. **Register a user** in the mobile app
2. **Check email verification** at http://localhost:8025
3. **Click verification link** in the email
4. **Login** with the verified user

### Making Changes

- **Backend changes**: Auto-reload enabled
- **Mobile changes**: Hot reload via Expo
- **Database changes**: Data persists in Docker volumes

### Useful Commands

```bash
# View all logs
./SoulTalk-Infra/scripts/logs.sh

# View specific service logs
./SoulTalk-Infra/scripts/logs.sh backend

# Stop everything
./SoulTalk-Infra/scripts/stop.sh

# Reset everything (fresh start)
./SoulTalk-Infra/scripts/reset.sh

# Check service status
docker-compose ps
```

## API Testing

The backend provides a Swagger UI for API testing:
- **API Docs**: http://localhost:8000/docs
- **Health Check**: http://localhost:8000/health

## Troubleshooting

### Services won't start
```bash
# Check Docker is running
docker version

# Reset everything
./SoulTalk-Infra/scripts/reset.sh
```

### Mobile app can't connect to backend
- Make sure CORS is enabled (it is by default)
- Check that backend is running: http://localhost:8000/health

### Email verification not working
- Check Mailhog: http://localhost:8025
- Emails appear instantly in development

## File Structure

```
SoulTalk/
├── SoulTalk-backend/     # FastAPI backend
├── SoulTalk-Mobile/      # React Native app
├── SoulTalk-Infra/       # Development infrastructure
│   ├── scripts/          # Helper scripts
│   └── README.md
├── docker-compose.yml    # All services
└── .env                  # Environment config
```

## Next Steps

- Add new API endpoints in `SoulTalk-backend/app/api/`
- Add new mobile screens in `SoulTalk-Mobile/src/screens/`
- Modify authentication flows in both backend and mobile app
- Use Mailhog for email testing

Keep it simple! 🚀