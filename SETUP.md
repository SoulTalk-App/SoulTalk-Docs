# SoulTalk Development Setup Guide

This guide will walk you through setting up the complete SoulTalk development environment with Keycloak authentication.

## 🎯 What You'll Build

By the end of this setup, you'll have:
- ✅ Keycloak authentication server running locally
- ✅ FastAPI backend with Keycloak integration
- ✅ PostgreSQL database for Keycloak
- ✅ Redis for caching and session management
- ✅ React Native mobile app with authentication flows
- ✅ Email testing with Mailhog
- ✅ Complete user registration, login, and password reset flows

## 📋 Prerequisites Checklist

Before starting, ensure you have:

- [ ] **Docker Desktop** installed and running
- [ ] **Docker Compose** (comes with Docker Desktop)
- [ ] **Node.js 18+** (for React Native development)
- [ ] **Python 3.11+** (optional, for backend development)
- [ ] **Git** for version control
- [ ] **Code Editor** (VS Code recommended)

### Optional for Mobile Development:
- [ ] **Expo CLI**: `npm install -g @expo/cli`
- [ ] **Expo Go** app on your phone (for testing)
- [ ] **iOS Simulator** (Mac only) or **Android Emulator**

## 🚀 Step-by-Step Setup

### Step 1: Project Structure Verification

Verify your project structure looks like this:
```
SoulTalk/
├── SoulTalk-backend/         # FastAPI backend
├── SoulTalk-Mobile/          # React Native app
├── SoulTalk-Infra/           # Infrastructure
│   ├── docker-compose.yml
│   ├── keycloak-realm-config.json
│   └── scripts/
├── SoulTalk-Docs/           # Documentation
└── README.md               # Project overview
```

### Step 2: Start Infrastructure Services

```bash
cd SoulTalk-Infra
./scripts/start.sh
```

**Expected Output:**
```
🚀 Starting SoulTalk Development Environment...
📦 Starting Docker services...
Creating soultalk-postgres ... done
Creating soultalk-redis    ... done
Creating soultalk-keycloak ... done
Creating soultalk-backend  ... done
Creating soultalk-mailhog  ... done
```

### Step 3: Wait for Services to Start

Monitor the startup process:
```bash
cd SoulTalk-Infra
docker-compose ps
```

Wait until all services show "healthy" status. Keycloak takes 2-3 minutes to fully start.

### Step 4: Verify Services

Check that all services are accessible:

- **Backend API**: http://localhost:8000/health
- **Keycloak Admin**: http://localhost:8080/admin (admin/admin)
- **Mailhog**: http://localhost:8025
- **API Documentation**: http://localhost:8000/docs

### Step 5: Set Up Mobile App

```bash
cd SoulTalk-Mobile
npm install
npm start
```

Follow the Expo instructions:
- Press `w` to open in web browser
- Press `i` for iOS simulator
- Press `a` for Android emulator
- Scan QR code with Expo Go app on your phone

### Step 6: Test Complete Authentication Flow

1. **Open the mobile app** (http://localhost:8081 for web)
2. **Register a new user**:
   - Fill in the registration form
   - Use any email address (emails are caught by Mailhog)
   - Password requirements: 8+ chars with uppercase, lowercase, number, special char
3. **Check email verification** at http://localhost:8025
4. **Click verification link** in the email
5. **Return to mobile app** and log in with your credentials
6. **Verify successful login** - you should see the home screen

## 🔍 Verification Checklist

After setup, verify each component:

### Infrastructure Services
- [ ] **PostgreSQL**: Running on port 5433
- [ ] **Redis**: Running on port 6379  
- [ ] **Keycloak**: Accessible at http://localhost:8080
- [ ] **Backend API**: Accessible at http://localhost:8000
- [ ] **Mailhog**: Accessible at http://localhost:8025

### Keycloak Configuration
- [ ] **SoulTalk realm**: Exists and configured
- [ ] **Backend client**: soultalk-backend configured
- [ ] **Mobile client**: soultalk-mobile configured  
- [ ] **User groups**: free-users, premium-users, admin-users created
- [ ] **Email verification**: Required for new users

### Mobile App
- [ ] **App starts**: Expo development server running
- [ ] **Authentication screens**: Login and register screens load
- [ ] **API connectivity**: App can communicate with backend
- [ ] **Token storage**: Secure token storage working

## 🧪 Test API Endpoints

Test the backend API directly:

```bash
# Test health endpoint
curl http://localhost:8000/health

# Register a new user
curl -X POST "http://localhost:8000/api/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123!",
    "first_name": "Test",
    "last_name": "User"
  }'

# After email verification, test login
curl -X POST "http://localhost:8000/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123!"
  }'

# Test protected endpoint (use access_token from login)
curl -X GET "http://localhost:8000/api/auth/me" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

## 🚨 Common Issues and Solutions

### Issue: Services Won't Start
**Symptoms**: Docker containers keep restarting
**Solution**:
```bash
cd SoulTalk-Infra
# Check logs for specific errors
docker-compose logs

# Reset everything
./scripts/reset.sh

# Check Docker memory allocation (should be 4GB+)
```

### Issue: Backend Can't Connect to Keycloak
**Symptoms**: Authentication endpoints return 500 errors
**Solution**:
```bash
cd SoulTalk-Infra
# Verify Keycloak is accessible from backend
docker-compose exec backend curl http://keycloak:8080/health/ready

# Check environment variables
docker-compose exec backend env | grep KEYCLOAK

# Restart backend
docker-compose restart backend
```

### Issue: Mobile App Can't Connect to Backend
**Symptoms**: Network errors in mobile app
**Solution**:
- **For web browser**: Use http://localhost:8000
- **For iOS Simulator**: Use http://localhost:8000
- **For Android Emulator**: Use http://10.0.2.2:8000
- **For physical device**: Use your computer's IP address

Find your IP:
```bash
# Mac
ipconfig getifaddr en0

# Linux  
hostname -I | awk '{print $1}'

# Windows
ipconfig | findstr IPv4
```

### Issue: Email Verification Not Working
**Symptoms**: No verification emails in Mailhog
**Solution**:
```bash
cd SoulTalk-Infra
# Check Keycloak SMTP configuration
# Go to http://localhost:8080/admin
# SoulTalk realm -> Realm Settings -> Email
# Should be configured to use Mailhog (host: mailhog, port: 1025)

# Check backend logs for email errors
docker-compose logs backend | grep -i email
```

## 📱 Mobile Development Tips

### Running on Physical Device
```bash
# Find your computer's IP address
ipconfig getifaddr en0  # Mac

# Update mobile app configuration if needed
# Check SoulTalk-Mobile/src/services/AuthService.ts
# Ensure API base URL uses your IP instead of localhost
```

### Debugging Authentication
```javascript
// Add to mobile app for debugging
console.log('Login request:', loginData);
console.log('Login response:', response);
console.log('Stored tokens:', await getTokens());
```

## 🔧 Development Commands

### Infrastructure Management
```bash
cd SoulTalk-Infra

# Start services
./scripts/start.sh

# View logs
./scripts/logs.sh [service-name]

# Stop services  
./scripts/stop.sh

# Reset everything (removes all data)
./scripts/reset.sh

# Check status
docker-compose ps
```

### Backend Development
```bash
cd SoulTalk-backend

# Install dependencies
pip install -r requirements.txt

# Run tests (if available)
pytest

# Check code formatting
black --check app/
```

### Mobile Development  
```bash
cd SoulTalk-Mobile

# Install dependencies
npm install

# Start development server
npm start

# Run type checking
npx tsc --noEmit

# Clear cache
npx expo start --clear
```

## 🎉 Success Verification

If everything is working correctly, you should be able to:

1. ✅ **Register** a new user via mobile app
2. ✅ **Receive verification email** in Mailhog  
3. ✅ **Click verification link** and verify account
4. ✅ **Login** with verified credentials
5. ✅ **Access protected routes** in mobile app
6. ✅ **View API documentation** at http://localhost:8000/docs
7. ✅ **See user profile** in mobile app

## 📚 Next Steps

After successful setup:

1. **Explore the codebase**: Understand the authentication flow
2. **Customize mobile UI**: Modify screens and components
3. **Add new API endpoints**: Extend backend functionality  
4. **Configure production settings**: Prepare for deployment
5. **Set up monitoring**: Add logging and health checks

## 🤝 Getting Help

If you encounter issues:

1. Check the [troubleshooting section](#-common-issues-and-solutions) above
2. Review service logs: `./SoulTalk-Infra/scripts/logs.sh [service]`
3. Verify all prerequisites are installed
4. Try the reset script: `./SoulTalk-Infra/scripts/reset.sh`

**You're ready to start building with SoulTalk! 🚀**