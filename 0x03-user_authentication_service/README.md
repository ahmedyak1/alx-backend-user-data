## Simple User authentication service

## Simple User authentication service


## Routes
GET /: returns the JSON payload with homepage content
POST /users: returns JSON payload of the form containing various user info
POST /sessions: returns JSON payload of the form containing login info
DELETE /sessions: deletes a user session based on the ID
GET /profile: returns A JSON payload containing the email if successful
POST /reset_password: returns A JSON payload containing the email & reset token if successful
PUT /reset_password: returns A JSON payload containing the email & message Password updated if successful

