# SoulTalk - Secure Authentication Platform

A modern authentication platform built with FastAPI, React Native, and Keycloak integration.

## 🏗️ Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Mobile App    │    │   Backend API   │    │    Keycloak     │
│  React Native   │◄──►│    FastAPI      │◄──►│  Auth Server    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │                       │
                                ▼                       ▼
                       ┌─────────────────┐    ┌─────────────────┐
                       │   PostgreSQL    │    │     Redis       │
                       │    Database     │    │     Cache       │
                       └─────────────────┘    └─────────────────┘
```

## 📁 Project Structure

```
SoulTalk/
├── SoulTalk-backend/     # FastAPI backend with Keycloak integration
├── SoulTalk-Mobile/      # React Native mobile app
├── SoulTalk-Infra/       # Docker infrastructure and scripts
│   ├── docker-compose.yml
│   ├── keycloak-realm-config.json
│   └── scripts/
├── SoulTalk-Docs/        # All documentation
└── README.md             # This file
```

## 🚀 Quick Start

### Prerequisites

- Docker Desktop (running)
- Node.js 18+ (for mobile development)
- Python 3.11+ (optional, for backend development)

### 1. Start Infrastructure

```bash
cd SoulTalk-Infra
./scripts/start.sh
```

### 2. Start Mobile App

```bash
cd SoulTalk-Mobile
npm install
npm start
```

### 3. Access Applications

- **Mobile App**: http://localhost:8081
- **Backend API**: http://localhost:8000/docs
- **Keycloak Admin**: http://localhost:8080/admin (admin/admin)
- **Email Testing**: http://localhost:8025

## 🔐 Authentication Features

### Implemented Features
- **User Registration** with email verification
- **Password-based Login** with JWT tokens
- **Token Refresh** for seamless experience
- **Password Reset** via email
- **Biometric Authentication** (mobile)
- **Role-based Access Control** with user groups

### Security Features
- **JWT Token Validation** with Keycloak
- **Secure Token Storage** on mobile
- **Automatic Token Refresh**
- **CORS Protection**
- **Rate Limiting**

## 📱 Mobile App

React Native app with complete authentication flows:
- Login/Register screens
- Email verification handling
- Biometric authentication support
- Secure token management
- Automatic session refresh

## 🔧 Backend API

FastAPI backend with Keycloak integration:
- RESTful authentication endpoints
- JWT token validation
- User profile management
- Password reset functionality
- Health checks and monitoring

## 🔑 Keycloak Configuration

- **Realm**: `soultalk`
- **Clients**: Backend and mobile clients configured
- **User Groups**: free-users, premium-users, admin-users
- **Email Verification**: Required for new users
- **Password Policy**: Strong password requirements

## 📚 Documentation

- **[Development Guide](DEVELOPMENT.md)** - Quick local setup (2 minutes)
- **[Setup Guide](SETUP.md)** - Detailed setup instructions
- **[Infrastructure Guide](INFRASTRUCTURE.md)** - Docker services and scripts
- **[Email Setup](setup-smtp.md)** - SMTP configuration for email verification

## 🛠️ Development

```bash
# Start all services
./SoulTalk-Infra/scripts/start.sh

# Start mobile app
cd SoulTalk-Mobile && npm start

# View logs
./SoulTalk-Infra/scripts/logs.sh

# Stop services
./SoulTalk-Infra/scripts/stop.sh
```

## 🧪 Testing Authentication

1. **Register** a new user in the mobile app
2. **Verify email** at http://localhost:8025
3. **Login** with verified credentials
4. **Test API** at http://localhost:8000/docs

## 🚨 Troubleshooting

### Services won't start
```bash
# Reset everything
./SoulTalk-Infra/scripts/reset.sh
```

### Mobile app connection issues
- Check CORS configuration in backend
- Verify backend is running: http://localhost:8000/health
- For physical device, update IP addresses in mobile app config

### Email verification not working
- Check Mailhog: http://localhost:8025
- Verify SMTP configuration in Keycloak admin console

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

---

**SoulTalk Team** - Building secure, scalable authentication experiences 🔐