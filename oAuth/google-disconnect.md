# Disconnect Google Services
### **What This Does**

**Disconnects Google:** Deletes the `GoogleCredentials` record for the user and revokes the access token with Google.

**Keeps Session:** Your login state remains intact—only the Google connection is removed.

**Simple UI:** Adds a button to your dashboard for easy access.
## Backend: Add Disconnect Google Endpoint
### Remove Credentials
Add an endpoint to remove the `GoogleCredentials` for the authenticated user and revoke the token.

**Revocation:** The `requests.post` to Google’s revoke endpoint ensures the access token is invalidated on Google’s side.\
**Error Handling:** Returns `200` even if no credentials exist (idempotent), but errors out with `500` if something unexpected happens.

```Python
# auth_app/views.py
from rest_framework import permissions, status
from rest_framework.response import Response
from rest_framework_simplejwt import authentication
from .models import GoogleCredentials
import requests

class DisconnectGoogleView(APIView)
    permission_classes = [permissions.IsAuthenticated]
    authentication_classes = [authentication.JWTAuthentication]

    def post(self, request):
        try:
            # Fetch the user's Google credentials
            credentials = GoogleCredentials.objects.get(user=request.user)
            access_token = credentials.access_token
            
            # Revoke the access token with Google
            revoke_url = 'https://oauth2.googleapis.com/revoke'
            revoke_response = requests.post(revoke_url, data={'token': access_token})
            if revoke_response.status_code != 200:
                print(f"Failed to revoke Google token: {revoke_response.text}")
                # Continue even if revocation fails—credential deletion is the priority
            
            # Delete the credentials from the database
            credentials.delete()
            print(f"Google credentials revoked and deleted for user: {request.user.username}")
            return Response({"message": "Google account disconnected successfully"}, status=status.HTTP_200_OK)
        
        except GoogleCredentials.DoesNotExist:
            print(f"No Google credentials found for user: {request.user.username}")
            return Response({"message": "No Google account connected"}, status=status.HTTP_200_OK)
        except Exception as e:
            print(f"Error disconnecting Google: {e}")
            return Response({"error": "Failed to disconnect Google"}, status=status.HTTP_500_INTERNAL_SERVER_ERROR)
  ```
### **Add to `urls.py`**
  ```Python
  # auth_app/urls.py
  from django.urls import path
  from . import views
  
  urlpatterns = [
      # ... your existing paths ...
      path('auth/google/disconnect/', views.disconnect_google, name='disconnect_google'),
  ]
  ```
## Frontend: Add Disconnect Button in Dashboard

### **Update `api.js`**

Assuming your dashboard is a protected route (`Dashboard.jsx`), add a button to call the disconnect endpoint.
```Javascript
// src/services/api.js
import axios from 'axios';

// Create api service
const api = axios.create({
    baseURL: import.meta.env.VITE_API_URL || 'http://localhost:8000/',
    withCredentials: true,
});

// JWT Interceptor
api.interceptors.request.use((config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// Existing functions...
export async function startGoogleOAuth() { /* ... */ }
export async function completeGoogleOAuth() { /* ... */ }

// --- New Disconnect Function ---
export async function disconnectGoogle() {
    try {
        const response = await api.post('auth/google/disconnect/');
        console.log(response.data.message);
        return response.data;
    } catch (error) {
        console.error('Failed to disconnect Google:', error.response?.data || error.message);
        throw error;
    }
}

export default api;
```
### **Update `Dashboard.jsx`**
Add a button to disconnect Google:

**State:** `message` shows feedback ("Google account disconnected successfully").\
**Protected Route:** Since your dashboard is already JWT-protected, the `Authorization` header will be sent automatically via the Axios interceptor.

```Javascript
// src/components/Dashboard.jsx (or wherever your dashboard lives)
import React, { useState } from 'react';
import { disconnectGoogle } from '../services/api';
import { useNavigate } from 'react-router-dom'; // Optional, if you want to redirect

function Dashboard() {
    const [message, setMessage] = useState('');
    const navigate = useNavigate(); // Optional

    const handleDisconnectGoogle = async () => {
        try {
            const response = await disconnectGoogle();
            setMessage(response.message); // Display success message
            // Optional: Refresh the page or update UI
            // window.location.reload();
        } catch (error) {
            setMessage('Failed to disconnect Google');
        }
    };

    return (
        <div>
            <h1>Dashboard</h1>
            {/* Your existing dashboard content */}
            <button onClick={handleDisconnectGoogle}>Disconnect Google</button>
            {message && <p>{message}</p>}
            {/* Optional: Your existing logout button */}
        </div>
    );
}

// Assuming this is protected with JWT
export default Dashboard;

```

## Verify It Works - SANITY CHECK
### **Backend:**
Ensure the new endpoint is added and the server is running.

**Test manually**

```zsh
curl -X POST http://localhost:8000/auth/google/disconnect/ -H "Authorization: Bearer <your-jwt-token>"
```

### **Frontend:**
Log in, go to the dashboard, and click the "Disconnect Google" button.

Check the console for logs and the UI for the success message.
### **Logs:**
**Backend:** Look for `INFO ... Google credentials revoked and deleted for user:`

**Frontend:** `console.log` should show "Google account disconnected successfully".
