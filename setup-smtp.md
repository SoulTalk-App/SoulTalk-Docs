# Email Setup for SoulTalk

Email verification is already configured with Mailhog for development. This guide shows additional options for production use.

## Current Setup (Development)

**Mailhog is already configured and working:**
- SMTP Server: Mailhog (automatic)
- Web UI: http://localhost:8025
- All emails are caught and displayed in the web interface
- No external SMTP configuration needed

## Production Options

### Option 1: Gmail SMTP Setup

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

## Development Testing (Already Configured)

**Mailhog is already set up in docker-compose.yml:**
- Container running automatically
- SMTP configured in Keycloak
- All verification emails appear at http://localhost:8025
- No additional configuration needed

## Option 2: SendGrid (Production-Ready)

1. Sign up for SendGrid free account
2. Create API key
3. Configure Keycloak SMTP:
   - **Host**: smtp.sendgrid.net
   - **Port**: 587
   - **Username**: apikey
   - **Password**: your-sendgrid-api-key
   - **From**: your-verified-sender@yourdomain.com

## Testing Email Verification

**Development (Current Setup):**
1. Register a new user via mobile app or API
2. Check Mailhog at http://localhost:8025 for verification email
3. Click the verification link in the email
4. Return to app and login - should work

**Production:**
1. Configure Gmail or SendGrid SMTP in Keycloak
2. Test with real email addresses
3. Verify emails are delivered to actual inboxes

## Current Status
- ✅ Keycloak realm configured for email verification
- ✅ User registration creates users with email verification required  
- ✅ Mailhog SMTP server configured and working
- ✅ Emails are being sent and caught by Mailhog

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