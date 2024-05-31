# Session Authentication

This project involves tasks for learning to authenticate a user through session authentication.

## Tasks To Complete

### 0. Et moi et moi et moi!

Copy all your work from the `0x06. Basic authentication` project into this new folder. In this version, you implemented Basic authentication to provide access to all User endpoints:

- `GET /api/v1/users`
- `POST /api/v1/users`
- `GET /api/v1/users/<user_id>`
- `PUT /api/v1/users/<user_id>`

### 1. Empty Session

Create a class `SessionAuth` in `api/v1/auth/session_auth.py` that inherits from `Auth`. For now, this class will be empty. This is the first step in creating a new authentication mechanism:

- Validate that everything inherits correctly without any overloading.
- Validate the "switch" by using environment variables.
- Update `api/v1/app.py` to use `SessionAuth` for the variable `auth` depending on the value of the environment variable `AUTH_TYPE`. If `AUTH_TYPE` is equal to `session_auth`:
  - Import `SessionAuth` from `api/v1/auth/session_auth`
  - Create an instance of `SessionAuth` and assign it to the variable `auth`
  - Otherwise, keep the previous mechanism.

### 2. Create a Session

Update the `SessionAuth` class:

- Create a class attribute `user_id_by_session_id` initialized as an empty dictionary.
- Create an instance method `def create_session(self, user_id: str = None) -> str:` that creates a Session ID for a user_id:
  - Return `None` if `user_id` is `None` or not a string.
  - Otherwise:
    - Generate a Session ID using the `uuid` module and `uuid4()` like `id` in `Base`.
    - Use this Session ID as the key of the dictionary `user_id_by_session_id` - the value for this key must be `user_id`.
    - Return the Session ID.
- The same `user_id` can have multiple Session IDs - indeed, the `user_id` is the value in the dictionary `user_id_by_session_id`.

### 3. User ID for Session ID

Update the `SessionAuth` class:

- Create an instance method `def user_id_for_session_id(self, session_id: str = None) -> str:` that returns a User ID based on a Session ID:
  - Return `None` if `session_id` is `None` or not a string.
  - Return the value (the User ID) for the key `session_id` in the dictionary `user_id_by_session_id`.
  - You must use `.get()` built-in for accessing a value in a dictionary based on key.

### 4. Session Cookie

Update `api/v1/auth/auth.py` by adding the method `def session_cookie(self, request=None):` that returns a cookie value from a request:

- Return `None` if `request` is `None`.
- Return the value of the cookie named `_my_session_id` from the request - the name of the cookie must be defined by the environment variable `SESSION_NAME`.
- You must use `.get()` built-in for accessing the cookie in the request cookies dictionary.
- You must use the environment variable `SESSION_NAME` to define the name of the cookie used for the Session ID.

### 5. Before Request

Update the `@app.before_request` method in `api/v1/app.py`:

- Add the URL path `/api/v1/auth_session/login/` to the list of excluded paths of the method `require_auth` - this route doesn't exist yet but should be accessible without authentication.
- If `auth.authorization_header(request)` and `auth.session_cookie(request)` return `None`, abort with a status code of 401.

### 6. Use Session ID for Identifying a User

Update the `SessionAuth` class:

- Create an instance method `def current_user(self, request=None):` (overload) that returns a User instance based on a cookie value:
  - You must use `self.session_cookie(...)` and `self.user_id_for_session_id(...)` to return the User ID based on the cookie `_my_session_id`.
  - Using this User ID, you will be able to retrieve a User instance from the database - you can use `User.get(...)` for retrieving a User from the database.

### 7. New View for Session Authentication

Create a new Flask view that handles all routes for the Session authentication:

- In the file `api/v1/views/session_auth.py`, create a route `POST /auth_session/login` (= `POST /api/v1/auth_session/login`):
  - Slash tolerant (`/auth_session/login` == `/auth_session/login/`).
  - You must use `request.form.get()` to retrieve `email` and `password` parameters.
  - If `email` is missing or empty, return the JSON `{ "error": "email missing" }` with a status code of 400.
  - If `password` is missing or empty, return the JSON `{ "error": "password missing" }` with a status code of 400.
  - Retrieve the User instance based on the email - you must use the class method `search` of `User` (same as the one used for `BasicAuth`).
  - If no User is found, return the JSON `{ "error": "no user found for this email" }` with a status code of 404.
  - If the password is not the one for the User found, return the JSON `{ "error": "wrong password" }` with a status code of 401 - you must use `is_valid_password` from the User instance.
  - Otherwise, create a Session ID for the User ID:
    - You must use `from api.v1.app import auth` - WARNING: please import it only where you need it - not on top of the file (can generate circular imports - and break the first tasks of this project).
    - You must use `auth.create_session(...)` for creating a Session ID.
    - Return the dictionary representation of the User - you must use `to_json()` method from User.
    - You must set the cookie to the response - you must use the value of the environment variable `SESSION_NAME` as the cookie name.
- In the file `api/v1/views/__init__.py`, you must add this new view at the end of the file.
- Now you have an authentication based on a Session ID stored in a cookie, perfect for a website (browsers love cookies).

### 8. Logout

Update the `SessionAuth` class by adding a new method `def destroy_session(self, request=None):` that deletes the user session / logout:

- If the request is `None`, return `False`.
- If the request doesn't contain the Session ID cookie, return `False` - you must use `self.session_cookie(request)`.
- If the Session ID of the request is not linked to any User ID, return `False` - you must use `self.user_id_for_session_id(...)`.
- Otherwise, delete the Session ID in `self.user_id_by_session_id` (as the key of this dictionary) and return `True`.

Update the file `api/v1/views/session_auth.py`, by adding a new route `DELETE /api/v1/auth_session/logout`:

- Slash tolerant.
- You must use `from api.v1.app import auth`.
- You must use `auth.destroy_session(request)` for deleting the Session ID contents in the request as a cookie:
  - If `destroy_session` returns `False`, abort with a status code of 404.
  - Otherwise, return an empty JSON dictionary with a status code of 200.

### 9. Expiration?

Now you have 2 authentication systems:

- Basic authentication.
- Session authentication.

Next, you will add an expiration date to a Session ID:

- Create a class `SessionExpAuth` that inherits from `SessionAuth` in the file `api/v1/auth/session_exp_auth.py`:
  - Overload the `def __init__(self):` method:
    - Assign an instance attribute `session_duration`:
      - To the environment variable `SESSION_DURATION` cast to an integer.
      - If this environment variable doesn't exist or can't be parsed to an integer, assign to 0.
  - Overload the `def create_session(self, user_id=None):` method:
    - Create a Session ID by calling `super()` - `super()` will call the `create_session()` method of `SessionAuth`.

### 10. Sessions in Database

Since the beginning, all Session IDs are stored in memory. This means, if your application stops, all Session IDs are lost. To avoid that, you will create a new authentication system based on Session IDs stored in a database (for us, it will be in a file, like User):

- Create a new model `UserSession` in `models/user_session.py` that inherits from `Base`:
  - Implement the `def __init__(self, *args: list, **kwargs: dict):` like in `User` but for these 2 attributes:
    - `user_id`: string.
    - `session_id`: string.
- Create a new authentication class `SessionDBAuth` in `api/v1/auth/session_db_auth.py` that inherits from `SessionExpAuth`.
