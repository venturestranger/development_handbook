# **API Development Guide**

###### Outline:

1. Folder Structure
2. Headers for PUT and POST Requests
3. Request Argument Guidelines
4. Authorization Middleware
5. Authorization Module



## Folder Structure

The project follows a structured hierarchy for modularity and scalability:

```
/ --- root ---

# Main entry point for connecting routers and defining the app factory for Gunicorn
- main.py

# Reusable configuration and utility items
- state.py

# Configuration classes with versioning
- config.py

# HTTP request preprocessing
- routers/
  ├─ v1/
  │   ├─ users.py

# Models and validation schemas for routers
- schemas/
  ├─ v1/
  │   ├─ users.py

# Data postprocessing from HTTP requests
- views/
  ├─ v1/
  │   ├─ users.py

# Functional endpoints for single-method actions
- handlers/
  ├─ v1/
  │   ├─ users.py

# Functional components for data postprocessing
- functional/
  ├─ v1/
  │   ├─ utils.py
  │   ├─ users.py

# Middleware definitions
- middlewares/
  ├─ v1/
  │   ├─ security.py
  │   ├─ session.py

# Temporary file storage
- tmp/
  ├─ tmp_1828x812.*

# Cached file storage
- cache/
  ├─ cache.db
  ├─ file_17XSDB18.*
```

---

### Router Model (HTTP Request Preprocessing)

Define routes using HTTP methods:

```python
@user_router.get("/", summary="Retrieve all users")
async def get_users():
    ...

@user_router.get("/{user_id}", summary="Retrieve a user by ID")
async def get_user(user_id: int):
    ...

@user_router.post("/", summary="Create a new user")
async def create_user(user: dict):
    ...
```

---

### View Model (HTTP Data Postprocessing)

Handle data postprocessing and application logic in static methods:

```python
class Users:
    @staticmethod
    def get(*args):  # Data retrieved from the HTTP request
        # Postprocessing logic here

    @staticmethod
    def post(*args):
        ...

    @staticmethod
    def delete(*args):
        ...

    @staticmethod
    def update(*args):
        ...
```

---

### Functional Model (Complex Postprocessing)

Isolate complex logic and algorithms into separate methods:

```python
class Users:
    @staticmethod
    def perform_action_x(*args):
        # Complex logic here

    @staticmethod
    def perform_action_y(*args):
        ...
```

---



## Headers for PUT and POST Requests

```json
{
    "location": "{config.LOCATION_BASE}/api/rest/v1/users/123",
    "cache-id": "cache_identifier",
    "cache-lt": "expiration_time"
}
```

---



## **Request Argument Guidelines**

- **GET**: Use **query parameters** for identifiers and filters.
- **POST**: 
  - Use query parameters for result-non-impacting fields (e.g., `sub`).
  - Include data in the **JSON body** or **post-form** for file uploads.
- **DELETE**: 
  - Use query parameters for identifiers only. Avoid filters.
- **PUT**: 
  - Use query parameters for identifiers.
  - Include all other data in the **JSON body** or **post-form** for file uploads.

---



## **Authorization Middleware**

Implements preflight checks to verify if the method and endpoint are allowed based on the `acs` field.

```python
from fastapi import Request, Response
from utils import config
import jwt

async def auth_middleware_v1(request: Request, call_next_handler):
    if config.AUTH_SKIP:
        return await call_next_handler(request)

    if str(request.url).endswith('/auth'):
        return await call_next_handler(request)

    try:
        auth_token = request.headers.get('Authorization', '').split()[-1].strip()
        payload = jwt.decode(auth_token, config.TOKEN_SECRET_KEY, algorithms=[config.ENCRYPTING_ALGORITHM])

        method = request.method
        action = request.url.path.split('/')[-1]

        if payload.get('iss') == config.TOKEN_ISSUER and (
            action in payload.get('permissions', {}).get(method, []) or payload.get('role') == 'admin'
        ):
            return await call_next_handler(request)
        else:
            return Response(content='Unauthorized', status_code=401)

    except Exception:
        return Response(content='Unauthorized', status_code=401)
```

---



## **Authorization Module**

### **Access vs. Refresh Tokens**
- **Access Token**: Contains API permissions (e.g., `{"GET": ["users", "wallets"]}`).
- **Refresh Token**: Used to regenerate access tokens when expired.

### **Token Fields**
- **v**: API version.
- **iss**: Issuer.
- **iat**: Issuance timestamp.
- **sub**: Client ID.
- **lvl**: Client role.
- **acs**: Access rules.

---

## **Authentication Endpoints**

### 1. **Login**

- **Endpoint**: `POST /auth/v1/login`  
- **Request**:
  ```json
  { "phone": "string" }
  ```
- **Response**:
  ```json
  { "message": "string", "verification_token": "string" }
  ```

### 2. **Refresh Token**

- **Endpoint**: `POST /auth/v1/refresh`  
- **Request**:
  ```json
  { "refresh_token": "string" }
  ```
- **Response**:
  ```json
  { "access_token": "string" }
  ```

### 3. **Register**

- **Endpoint**: `POST /auth/v1/register`  
- **Request**:
  ```json
  { "phone": "string" }
  ```
- **Response**:
  ```json
  { "message": "string", "verification_token": "string" }
  ```

### 4. **Verify Code**

- **Endpoint**: `POST /auth/v1/verify`  
- **Request**:
  ```json
  { "code": "string", "verification_token": "string" }
  ```
- **Response**:
  ```json
  {
    "message": "string",
    "access_token": "string",
    "refresh_token": "string"
  }
  ```

---

#### HTTP Status Codes  

- `200 OK`: Operation successful.
- `400 Bad Request`: Wrong request format.
- `401 Unauthorized`: 
  - At `login` - wrong credentials.
  - At `verify` - wrong code or verification token.
  - At `refresh` - wrong refresh token.
- `403 Forbidden`: 
  - At `register` - phone is already in use.

`message` has the same content as of the status code (except for `refresh` endpoint)
