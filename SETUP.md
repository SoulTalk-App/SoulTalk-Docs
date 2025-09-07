# SoulTalk Development Setup Guide

This guide will walk you through setting up the complete SoulTalk development environment with Keycloak authentication.

## 🎯 What You'll Build

By the end of this setup, you'll have:
- ✅ Keycloak authentication server running locally
- ✅ FastAPI backend with Keycloak integration
- ✅ PostgreSQL database for data persistence
- ✅ Redis for caching and session management
- ✅ React Native mobile app with authentication flows
- ✅ Complete user registration, login, and password reset flows

## 📋 Prerequisites Checklist

Before starting, ensure you have:

- [ ] **Docker Desktop** installed and running
- [ ] **Docker Compose** (comes with Docker Desktop)
- [ ] **Node.js 18+** (for React Native development)
- [ ] **Python 3.11+** (for backend development)
- [ ] **Git** for version control
- [ ] **Code Editor** (VS Code recommended)

### Optional for Mobile Development:
- [ ] **Expo CLI**: `npm install -g @expo/cli`
- [ ] **Expo Go** app on your phone (for testing)
- [ ] **iOS Simulator** (Mac only) or **Android Emulator**

## 🚀 Step-by-Step Setup

### Step 1: Project Setup

```bash
# Clone the repository (or create the directory structure)
cd Desktop
mkdir SoulTalk
cd SoulTalk

# Copy all the project files you've created
# The structure should look like:
# SoulTalk/
# ├── SoulTalk-backend/
# ├── SoulTalk-Mobile/
# ├── SoulTalk-Infra/
# ├── docker-compose.yml
# ├── nginx.conf
# ├── .env
# └── keycloak-realm-config.json
```

### Step 2: Environment Configuration

```bash
# Make sure your .env file has the correct values
cat > .env << 'EOF'
# Database Configuration
DATABASE_URL=postgresql://postgres:password@localhost:5432/soultalk
POSTGRES_DB=soultalk
POSTGRES_USER=postgres
POSTGRES_PASSWORD=password

# Keycloak Configuration
KEYCLOAK_SERVER_URL=http://localhost:8080
KEYCLOAK_REALM=soultalk
KEYCLOAK_CLIENT_ID=soultalk-backend
KEYCLOAK_CLIENT_SECRET=development-secret-123
KEYCLOAK_ADMIN_USERNAME=admin
KEYCLOAK_ADMIN_PASSWORD=admin

# Mobile App Keycloak Client
KEYCLOAK_MOBILE_CLIENT_ID=soultalk-mobile
KEYCLOAK_MOBILE_CLIENT_SECRET=mobile-development-secret-123

# Redis Configuration
REDIS_URL=redis://localhost:6379

# JWT Configuration
JWT_SECRET=development-jwt-secret-key-change-in-production
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=15
REFRESH_TOKEN_EXPIRE_DAYS=30

# Application Configuration
APP_NAME=SoulTalk
APP_VERSION=1.0.0
DEBUG=true
ENVIRONMENT=development

# CORS Configuration
CORS_ORIGINS=["http://localhost:3000", "http://127.0.0.1:3000"]

# Security
SECRET_KEY=development-secret-key-change-in-production
ALGORITHM=HS256

# Rate Limiting
RATE_LIMIT_PER_MINUTE=60
RATE_LIMIT_BURST=10

# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json
EOF
```

### Step 3: Start the Backend Services

```bash
# Start all backend services with Docker Compose
docker-compose up -d

# Verify services are starting
docker-compose ps

# Follow the logs to see startup progress
docker-compose logs -f
```

**Expected Output:**
```
Creating soultalk-postgres ... done
Creating soultalk-redis    ... done
Creating soultalk-keycloak ... done
Creating soultalk-backend  ... done
Creating soultalk-nginx    ... done
```

### Step 4: Wait for Keycloak to Initialize

Keycloak takes 2-3 minutes to fully start. Monitor the logs:

```bash
# Watch Keycloak logs until you see "Keycloak ... started"
docker-compose logs -f keycloak
```

**Look for this line:**
```
soultalk-keycloak | ... Keycloak 23.0.0 started in 45.123s
```

### Step 5: Import Keycloak Realm Configuration

```bash
# Get admin access token
ADMIN_TOKEN=$(curl -s -X POST 'http://localhost:8080/realms/master/protocol/openid-connect/token' \
  -d 'grant_type=password&username=admin&password=admin&client_id=admin-cli' | \
  python3 -c "import sys, json; print(json.load(sys.stdin)['access_token'])")

# Import the SoulTalk realm
curl -X POST "http://localhost:8080/admin/realms" \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d @keycloak-realm-config.json

# Verify realm was created
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  "http://localhost:8080/admin/realms/soultalk" | python3 -m json.tool
```

### Step 6: Verify Backend API

```bash
# Test health endpoint
curl http://localhost:8000/health

# Test API documentation
open http://localhost:8000/docs
```

**Expected Response:**
```json
{
  "status": "healthy",
  "service": "SoulTalk",
  "version": "1.0.0",
  "environment": "development"
}
```

### Step 7: Set Up Mobile App

```bash
cd SoulTalk-Mobile

# Install dependencies
npm install

# Start Expo development server
npx expo start

# Follow the QR code or press 'i' for iOS simulator, 'a' for Android emulator
```

### Step 8: Test the Complete Authentication Flow

1. **Open the mobile app** on your device or simulator
2. **Register a new user:**
   - Fill in the registration form
   - Use a real email (you'll need to verify it)
   - Password must meet requirements (8+ chars, uppercase, lowercase, number, special char)
3. **Check your email** for the verification link
4. **Click the verification link** to verify your account
5. **Return to the app** and log in with your credentials
6. **Test biometric authentication** (if available on your device)

## 🔍 Verification Checklist

After setup, verify each component:

### Backend Services
- [ ] **PostgreSQL**: `docker-compose exec postgres psql -U postgres -d soultalk -c "\dt"`
- [ ] **Redis**: `docker-compose exec redis redis-cli ping`
- [ ] **Keycloak**: Visit http://localhost:8080/admin (admin/admin)
- [ ] **Backend API**: Visit http://localhost:8000/docs
- [ ] **Nginx**: `curl -I http://localhost/health`

### Keycloak Configuration
- [ ] **Realm exists**: SoulTalk realm visible in admin console
- [ ] **Clients configured**: soultalk-backend and soultalk-mobile clients
- [ ] **Groups created**: free-users, premium-users, admin-users
- [ ] **Roles assigned**: user, premium, admin roles

### Mobile App
- [ ] **App starts**: Expo development server running
- [ ] **Authentication screens**: Login and register screens load
- [ ] **API connectivity**: App can communicate with backend
- [ ] **Token storage**: Secure token storage working

## 🧪 Test Authentication Flow

### Backend API Testing

```bash
# Test user registration
curl -X POST "http://localhost:8000/api/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123!",
    "first_name": "Test",
    "last_name": "User"
  }'

# Test user login (after email verification)
curl -X POST "http://localhost:8000/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123!"
  }'

# Test protected endpoint (use access_token from login response)
curl -X GET "http://localhost:8000/api/auth/me" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Mobile App Testing

1. **Registration Flow**:
   - Enter user details
   - Submit registration
   - Check for success message
   - Verify email verification requirement

2. **Login Flow**:
   - Enter credentials
   - Submit login
   - Verify token storage
   - Check navigation to home screen

3. **Token Management**:
   - Test automatic token refresh
   - Test logout functionality
   - Verify secure token storage

## 🚨 Common Issues and Solutions

### Issue: Keycloak Won't Start
**Symptoms**: Container keeps restarting
**Solution**:
```bash
# Check logs for specific error
docker-compose logs keycloak

# Common fix: Increase Docker memory allocation
# Docker Desktop -> Settings -> Resources -> Memory -> 4GB+

# Clean restart
docker-compose down -v
docker-compose up -d
```

### Issue: Backend Can't Connect to Keycloak
**Symptoms**: Authentication endpoints return 500 errors
**Solution**:
```bash
# Verify Keycloak is accessible from backend container
docker-compose exec backend curl http://keycloak:8080/health

# Check environment variables
docker-compose exec backend env | grep KEYCLOAK

# Restart backend service
docker-compose restart backend
```

### Issue: Mobile App Can't Connect to API
**Symptoms**: Network errors in mobile app
**Solution**:
```bash
# For iOS Simulator, use localhost
# For Android Emulator, use 10.0.2.2
# For physical device, use your computer's IP

# Find your IP
ipconfig getifaddr en0  # Mac
ip route get 1 | awk '{print $7}'  # Linux

# Update app.json with correct IP
```

### Issue: Email Verification Not Working
**Symptoms**: No verification emails received
**Solution**:
```bash
# Configure SMTP in Keycloak Admin Console
# Realm Settings -> Email
# Use Gmail SMTP or other email provider

# For development, check Keycloak logs for email content
docker-compose logs keycloak | grep -i email
```

## 📱 Mobile Development Tips

### Running on Physical Device
```bash
# Make sure your phone and computer are on the same network
# Update app.json with your computer's IP address
{
  "extra": {
    "keycloakConfig": {
      "url": "http://YOUR_IP:8080",
      "realm": "soultalk",
      "clientId": "soultalk-mobile"
    },
    "apiConfig": {
      "baseUrl": "http://YOUR_IP:8000/api"
    }
  }
}
```

### Debugging Authentication
```javascript
// Add to AuthService.ts for debugging
console.log('Token request:', tokenRequest);
console.log('Token response:', tokenResponse);
console.log('Stored token:', await SecureStore.getItemAsync('access_token'));
```

## 🔐 Security Notes

### Development Security
- Default passwords are for development only
- Change all secrets in production
- Enable HTTPS in production
- Use proper CORS origins

### Production Preparation
```bash
# Generate secure secrets
openssl rand -hex 32  # For JWT_SECRET
openssl rand -hex 32  # For SECRET_KEY
openssl rand -hex 16  # For client secrets
```

## 📚 Next Steps

After successful setup:

1. **Explore the Code**: Understand the authentication flow
2. **Customize UI**: Modify mobile app screens
3. **Add Features**: Extend API with new endpoints
4. **Production Setup**: Use Terraform for cloud deployment
5. **Monitoring**: Set up logging and monitoring

## 🤝 Getting Help

If you encounter issues:
1. Check the troubleshooting section above
2. Review Docker and service logs
3. Verify all environment variables
4. Check network connectivity between services

## 🎉 Success!

If you've completed all steps successfully, you now have:
- A fully functional Keycloak authentication system
- A FastAPI backend with JWT validation
- A React Native mobile app with complete auth flows
- A production-ready development environment

**You're ready to start building SoulTalk! 🚀**