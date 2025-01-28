# **API Development Guide**

###### Outline:

1. Folder Structure
2. Headers for PUT and POST Requests
3. Request Argument Guidelines
4. Filters
5. Sorting
6. Projection
7. Authorization Middleware
8. Authorization Module



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



## Filters

You can retrieve data by applying filters, which allow you to specify criteria for the fields and ranges of data:

- **Field-specific filters**: 
  Example: `/api/rest/v1/users?name=Jon&surname=Snow` 
  Retrieves users with the specified `name` and `surname`.

- **Range filters**: 
  Example: `/api/rest/v1/users?age=12$24` 
  Retrieves users within the age range of [12, 24].



## Sorting

You can sort the data by passing field names along with sorting orders:

- **Sorting syntax**: 
  Example: `/api/rest/v1/users?age=12&sort=name$-1,surname$1` 
  Explanation:  
  - `name$-1`: Sorts by the `name` field in descending order.  
  - `surname$1`: Sorts by the `surname` field in ascending order.  

The `sort` parameter accepts multiple fields, separated by `,`, each followed by `$1` (ascending) or `$-1` (descending).



## Projection

By default, every API request returns the complete model of the item. However, you can use projection to retrieve only specific fields of the model:

- **Projection syntax**: 
  Example: `/api/rest/v1/users?age=12&proj=name,surname` 
  Explanation:  
  - Retrieves only the `id`, `name`, and `surname` fields for users matching the specified `age` filter.  

The `proj` parameter allows you to list the desired fields, separated by `,`. This minimizes the data returned in the response, focusing only on the fields you need.



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
          action in payload.get('acs', {}).get(method, []) or 
          payload.get('lvl') == 'admin'
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

---

# README

## **Asset Service**

The **Asset Service** manages all uploaded files and supports the following operations:  
- **GET**: Retrieve a file by its ID.  
- **POST**: Upload a file and receive a file ID.  
- **DELETE**: Remove a file by its ID.  
 
**API Specifications**:  
- **GET**:  
  Consumes an HTTP request with a query parameter `id` (file ID).  
- **POST**:  
  Consumes a form-data HTTP request with the file attached and returns a JSON response:  
  ```json
  { "id": "unique-file-id" }
  ```
- **DELETE**:  
  Consumes an HTTP request with a query parameter `id` (file ID) to delete the file.

**Status Codes**:  
- `403`: Unauthorized (e.g., invalid token).  
- `422`: Unprocessable Entity (e.g., invalid file format or size).  
- `200`: OK (operation completed successfully).  
- `503`: Internal error (e.g., unexpected failure).

---

## **Message Service**

The **Message Service** is responsible for:  
1. **Sending Push Notifications**: Using Firebase Cloud Messaging.  
2. **Sending Verification Codes**: Through WhatsApp and Telegram.

### **Architectural Design**  
- **Agents Directory**:  
  Located at `/agents/v1` and contains the following folders:  
  - `/telegram`: Implements `TelegramAgent` for sending verification codes via Telegram.  
  - `/whatsapp`: Implements `WhatsappAgent` for sending verification codes via WhatsApp.  
  - `/firebase`: Implements `FirebaseAgent` for pushing notifications.  

- Each agent:  
  - Implements a method to send notifications or verification codes via its corresponding API.  
  - Listens to RabbitMQ signals, as defined in `/functional/v1`.  
  - Can also be manually triggered via handlers.

- **Router Preprocessing**:  
  Endpoints are preprocessed in `/router/v1` before being passed to handlers.

### **API Endpoints**  
- **POST Requests**:  
  - **Request Body**:  
    ```json
    { 
      "id": "user-identifier",
      "message": "message-content"
    }
    ```  
  - **Status Codes**:  
    - `403`: Unauthorized (e.g., invalid token).  
    - `422`: Unprocessable Entity (e.g., invalid request format).  
    - `200`: OK (operation completed successfully).  
    - `503`: Internal error (e.g., unsuccessful operation).

---

## **Streaming Service**

The **Streaming Service** handles WebSocket connections and provides real-time communication capabilities.  
### **Key Features**  
1. **WebSocket Support**:  
   - Handles WebSocket connections for chat communication.  
   - Sends messages in real-time to users specified in RabbitMQ messages.

2. **Directory Organization**:  
   - **Handlers**: Located in `/handlers/v1`, where the WebSocket logic is implemented.  
   - **Routers**: Located in `/routers/v1`, responsible for preprocessing and handling HTTP requests.  
   - **RabbitMQ Listener**: Defined in `/functional/v1`, listens for messages and triggers the appropriate handlers.

3. **WebSocket Message Format**:  
   - Follows the model architecture specified in the documentation.  

4. **Status Code Specification**:  
   - This service does not return traditional HTTP status codes as it operates via WebSockets.  
   - Invalid connections are handled by disconnecting clients.  

---

Each service is designed for clarity, scalability, and efficiency, with clear separation of concerns and modularity across the architecture.
