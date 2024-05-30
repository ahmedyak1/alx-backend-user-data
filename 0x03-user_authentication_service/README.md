## Simple User Authentication Service

### Files
- `user.py`: User model.
- `app.py`: Entry point of the API.
- `auth.py`: Authentication model.

### Routes

GET /: returns the JSON payload with homepage content
POST /users: returns JSON payload of the form containing various user info
POST /sessions: returns JSON payload of the form containing login info
DELETE /sessions: deletes a user session based on the ID
GET /profile: returns A JSON payload containing the email if successful
POST /reset_password: returns A JSON payload containing the email & reset token if successful
PUT /reset_password: returns A JSON payload containing the email & message Password updated if successful

### Run
Use the following command to start the API server:
```bash
API_HOST=0.0.0.0 API_PORT=5000 python3 -m app
