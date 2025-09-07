# SoulTalk - Complete Application with Keycloak Authentication

SoulTalk is a modern application built with FastAPI backend, React Native mobile app, and Keycloak for authentication and authorization.

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
├── SoulTalk-backend/       # FastAPI backend with Keycloak integration
├── SoulTalk-Mobile/        # React Native mobile app
├── SoulTalk-Infra/         # Infrastructure code (Terraform, K8s, CI/CD)
├── SoulTalk-Docs/          # Documentation and guides
├── docker-compose.yml      # Local development environment
├── nginx.conf             # Nginx configuration
├── .env                   # Environment variables
└── keycloak-realm-config.json  # Keycloak realm configuration
```

## 🚀 Quick Start

### Prerequisites

- Docker and Docker Compose
- Node.js 18+ (for mobile development)
- Python 3.11+ (for backend development)
- Expo CLI (for React Native)

### 1. Clone and Setup Environment

```bash
git clone <repository-url>
cd SoulTalk
cp .env.example .env
# Edit .env file with your configuration
```

### 2. Start Development Environment

```bash
# Start all services (Keycloak, PostgreSQL, Redis, Backend)
docker-compose up -d

# Wait for Keycloak to fully start (2-3 minutes)
docker-compose logs -f keycloak
```

### 3. Configure Keycloak Realm

```bash
# Import realm configuration
curl -X POST "http://localhost:8080/admin/realms" \
  -H "Authorization: Bearer $(curl -s -X POST 'http://localhost:8080/realms/master/protocol/openid-connect/token' \
    -d 'grant_type=password&username=admin&password=admin&client_id=admin-cli' | jq -r '.access_token')" \
  -H "Content-Type: application/json" \
  -d @keycloak-realm-config.json
```

### 4. Setup Mobile App

```bash
cd SoulTalk-Mobile
npm install
npx expo start
```

### 5. Access Applications

- **Keycloak Admin Console**: http://localhost:8080/admin (admin/admin)
- **Backend API Docs**: http://localhost:8000/docs
- **Mobile App**: Use Expo Go app on your phone or simulator

## 🔐 Authentication Flow

### Password Grant Flow (As Requested)

```mermaid
sequenceDiagram
    participant MA as Mobile App
    participant API as Backend API
    participant KC as Keycloak
    participant DB as Database

    MA->>API: POST /api/auth/login {email, password}
    API->>KC: Validate credentials
    KC-->>API: Return tokens (access + refresh)
    API-->>MA: Return tokens
    MA->>MA: Store tokens securely
    
    Note over MA,DB: Subsequent API calls
    MA->>API: API call with Bearer token
    API->>KC: Validate token
    KC-->>API: Token valid + user info
    API->>DB: Process request
    DB-->>API: Return data
    API-->>MA: Return response
```

### Token Management

- **Access Token**: Short-lived (15 minutes), used for API requests
- **Refresh Token**: Long-lived (30 days), used to get new access tokens
- **Automatic Refresh**: Mobile app automatically refreshes tokens
- **Biometric Login**: Optional biometric authentication for enhanced UX

## 📱 Mobile App Features

### Authentication Screens
- **Login Screen**: Email/password with biometric option
- **Registration Screen**: Complete user registration flow
- **Forgot Password**: Password reset via email
- **Email Verification**: Email verification flow

### Security Features
- **Secure Token Storage**: Uses Expo SecureStore
- **Biometric Authentication**: Face ID/Touch ID support
- **Automatic Token Refresh**: Seamless token management
- **Session Management**: Proper logout and session handling

## 🔧 Backend API Features

### Authentication Endpoints
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login (password grant)
- `POST /api/auth/logout` - User logout
- `POST /api/auth/refresh` - Token refresh
- `GET /api/auth/me` - Current user info
- `POST /api/auth/reset-password` - Password reset

### Security Features
- **JWT Token Validation**: Full Keycloak JWT validation
- **Role-Based Access Control**: User roles and groups
- **Rate Limiting**: Request rate limiting
- **CORS Protection**: Configurable CORS policies
- **Security Logging**: Comprehensive security event logging

## 🔑 Keycloak Configuration

### Realm Settings
- **Realm Name**: `soultalk`
- **User Registration**: Enabled
- **Email Verification**: Required
- **Password Policy**: Strong password requirements
- **Brute Force Protection**: Enabled

### Client Configuration

#### Backend Client (`soultalk-backend`)
- **Client Type**: Confidential
- **Authentication Flow**: Standard + Direct Access Grants
- **Service Accounts**: Enabled (for admin operations)

#### Mobile Client (`soultalk-mobile`)
- **Client Type**: Confidential
- **Authentication Flow**: Standard + Direct Access Grants
- **Public Client**: No (for enhanced security)

### User Groups and Roles
- **free-users**: Basic user access
- **premium-users**: Premium features access
- **admin-users**: Administrative access

## 🏭 Production Deployment

### Infrastructure Components

#### Terraform (AWS)
```bash
cd SoulTalk-Infra/terraform
terraform init
terraform plan -var-file="production.tfvars"
terraform apply
```

#### Kubernetes
```bash
cd SoulTalk-Infra/k8s
kubectl apply -f namespace.yaml
kubectl apply -f secrets-template.yaml  # After configuring secrets
kubectl apply -f keycloak-deployment.yaml
kubectl apply -f backend-deployment.yaml
```

#### CI/CD Pipeline
- **GitHub Actions**: Automated testing and deployment
- **Security Scanning**: Trivy vulnerability scanning
- **Multi-environment**: Staging and Production deployments

### Environment Configuration

#### Development
```env
KEYCLOAK_SERVER_URL=http://localhost:8080
DATABASE_URL=postgresql://postgres:password@localhost:5432/soultalk
```

#### Production
```env
KEYCLOAK_SERVER_URL=https://auth.soultalk.com
DATABASE_URL=postgresql://user:pass@prod-db:5432/soultalk
KC_HOSTNAME_STRICT_HTTPS=true
KC_PROXY=edge
```

## 🧪 Testing

### Backend Tests
```bash
cd SoulTalk-backend
pytest --cov=app tests/
```

### Mobile Tests
```bash
cd SoulTalk-Mobile
npm test
npm run type-check
npm run lint
```

### Integration Tests
```bash
# Start test environment
docker-compose -f docker-compose.test.yml up -d

# Run integration tests
pytest tests/integration/
```

## 🔧 Development Guidelines

### Backend Development
1. **Code Style**: Use Black, isort, and flake8
2. **Type Hints**: Full type annotations required
3. **Testing**: Unit and integration tests for all endpoints
4. **Security**: Never log secrets, validate all inputs

### Mobile Development
1. **TypeScript**: Strict TypeScript configuration
2. **Component Testing**: Test all authentication components
3. **Security**: Secure storage for tokens, proper error handling
4. **UX**: Loading states, offline handling, biometric integration

### Security Best Practices
1. **Token Handling**: Short-lived access tokens, secure refresh
2. **API Security**: Rate limiting, input validation, HTTPS only
3. **Mobile Security**: Certificate pinning, root detection
4. **Keycloak Security**: Strong password policies, brute force protection

## 📊 Monitoring and Logging

### Application Monitoring
- **Health Checks**: Kubernetes readiness and liveness probes
- **Metrics**: Application performance metrics
- **Logging**: Structured JSON logging with correlation IDs

### Security Monitoring
- **Authentication Events**: All auth events logged to Keycloak
- **Failed Attempts**: Brute force detection and alerting
- **Token Usage**: Token validation and refresh monitoring

## 🚨 Troubleshooting

### Common Issues

#### Keycloak Connection Issues
```bash
# Check Keycloak health
curl http://localhost:8080/health

# Verify realm configuration
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  http://localhost:8080/admin/realms/soultalk
```

#### Mobile App Issues
```bash
# Clear Expo cache
expo start --clear

# Check network connectivity
npx expo install @react-native-community/netinfo
```

#### Backend API Issues
```bash
# Check database connectivity
docker-compose exec backend python -c "from app.database import engine; print(engine.url)"

# Verify Keycloak integration
docker-compose logs backend | grep keycloak
```

### Development Reset
```bash
# Complete environment reset
docker-compose down -v
docker-compose up -d
# Wait 3 minutes for Keycloak startup
# Re-import realm configuration
```

## 📚 Additional Resources

- [Keycloak Documentation](https://www.keycloak.org/documentation)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [React Native Documentation](https://reactnative.dev/)
- [Expo Documentation](https://docs.expo.dev/)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**SoulTalk Team** - Building secure, scalable authentication experiences with Keycloak 🔐