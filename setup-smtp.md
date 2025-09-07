# Setting Up Email Verification for SoulTalk

## Option 1: Gmail SMTP Setup (Recommended for Development)

### Step 1: Get Gmail Credentials
1. Go to your Google Account: https://myaccount.google.com/
2. Navigate to "Security" → "2-Step Verification" (enable if not already)
3. Go to "App passwords" 
4. Generate a new app password for "Mail"
5. Copy the 16-character password (e.g., "abcd efgh ijkl mnop")

### Step 2: Configure Keycloak SMTP
1. Open Keycloak Admin Console: http://localhost:8080/admin
2. Login with: admin/admin
3. Go to "SoulTalk" realm
4. Navigate to "Realm Settings" → "Email" tab
5. Fill in:
   - **Host**: smtp.gmail.com
   - **Port**: 587
   - **From Display Name**: SoulTalk
   - **From**: noreply@yourdomain.com (or your Gmail address)
   - **Enable StartTLS**: ON
   - **Enable Authentication**: ON
   - **Username**: your-gmail@gmail.com
   - **Password**: your-16-character-app-password

6. Click "Test connection" to verify
7. Click "Save"

## Option 2: Use Development Email Service

### Using Mailhog (Email Testing Tool)
```bash
# Add to docker-compose.yml
mailhog:
  image: mailhog/mailhog:latest
  container_name: soultalk-mailhog
  ports:
    - "1025:1025"  # SMTP
    - "8025:8025"  # Web UI
  networks:
    - soultalk-network
```

Then configure Keycloak SMTP:
- **Host**: mailhog
- **Port**: 1025
- **Enable StartTLS**: OFF
- **Enable Authentication**: OFF
- **From**: noreply@soultalk.local

View emails at: http://localhost:8025

## Option 3: Use SendGrid (Production-Ready)

1. Sign up for SendGrid free account
2. Create API key
3. Configure Keycloak SMTP:
   - **Host**: smtp.sendgrid.net
   - **Port**: 587
   - **Username**: apikey
   - **Password**: your-sendgrid-api-key
   - **From**: your-verified-sender@yourdomain.com

## Testing Email Verification

After SMTP is configured:

1. Register a new user via API or mobile app
2. Check your email for verification link
3. Click the verification link
4. Try logging in - it should work now

## Current Status
- ✅ Keycloak realm configured for email verification
- ✅ User registration creates users with email verification required  
- ❌ SMTP server credentials missing
- ❌ Emails not being sent

## Quick Test Commands

```bash
# Register a user (will require email verification)
curl -X POST "http://localhost:8000/api/auth/register" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "password": "TestPass123!",
    "first_name": "Test",
    "last_name": "User"
  }'

# Try to login (will fail until email is verified)
curl -X POST "http://localhost:8000/api/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com", 
    "password": "TestPass123!"
  }'
```