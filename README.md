# DJANGOOGLE CALENDAR

**Backend** - Django / Django Rest Framework

**Frontend** - React and Javascript: Handles user interface, redirects, and sending data to the backend.

**Google** - The identity provider and resource server - provides authentication and calendar data

## Phase I: Prerequisites - Start here!

### 1. Google Cloud Console Setup:
1. Go to the Google Cloud Console.
2. Create a new Project (or use an existing one).
3. Enable APIs: Find and enable the "Google Calendar API" (and any other Google APIs you need scopes for).
4. Create Credentials:
   - Go to "APIs & Services" -> "Credentials".
   - Click "Create Credentials" -> "OAuth client ID".
   - Choose "Web application" as the application type.
   - Give it a name ("My Django App Web Client").
5. Add **Authorized JavaScript origins**: Enter the URL of your frontend application (e.g., `http://127.0.0.1:5173` for local development, or your domain, `https://yourapp.com` for production). This is used for `CORS`.
6. Add **Authorized redirect URIs**:
   - Enter the **exact** backend URL where Google should send the user back after they log in and consent. This URL will point to your `GoogleLoginCallbackView`.
   - (Example: `http://127.0.0.1:8000/api/auth/google/callback/`)
   - **Make sure the path matches your Django `urls.py`.**
8. Click "**Create**".
   Google will give you a `Client ID` and a `Client Secret`. Copy these **immediately** and **_keep the Client Secret secure_**!

### 2. Django Project Setup

1. Install Django and Django REST Framework (DRF).
2. Set up authentication for your application (the code uses JWTAuthentication, so you need djangorestframework-simplejwt or similar configured).
3. Set up Django sessions (the code uses request.session).
4. Create a Django model to store the Google credentials (`GoogleCredentials`). It needs fields for:\
`user` (**ForeignKey** to your `User` model), `access_token`, `refresh_token`, `expires_at`, `token_uri`, `client_id`, `client_secret`, `scopes`.

### 3. App Configuration
Store the `Client ID`, `Client Secret`, the **exact** `Redirect URI` you registered, and the desired Google API `scopes` securely in your Django `settings.py` or environment variables. The code accesses these via `settings.VARIABLE_NAME`.\
Example settings.py entries:
``` Python
GOOGLE_OAUTH2_CLIENT_ID = 'YOUR_CLIENT_ID.apps.googleusercontent.com'
GOOGLE_OAUTH2_CLIENT_SECRET = 'YOUR_CLIENT_SECRET'
GOOGLE_OAUTH2_REDIRECT_URI = 'http://127.0.0.1:8000/api/auth/google/callback/' # Must match Google Console
GOOGLE_CALENDAR_SCOPES = ['https://www.googleapis.com/auth/calendar'] # Or .readonly, .events etc.
```
