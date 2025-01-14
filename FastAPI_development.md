# API DEVELOPMENT NOTES



###### Folder's hierarchy

```python
/ --- root ---

# Connected routers, defined app_factory for gunicorn
- /main.py

# Defined reusable configuration and other utility items
- /state.py

# Defined configuration classes (with versioning)
- /config.py

# Defined preprocessing of HTTP packages
- /routers/
-- /v1/
--- /users.py

# Defined models' validation schemas (for routers)
- /schemas/
-- /v1/
--- /users.py

# Defined postprocessing of data from HTTP packages 
- /views/
-- /v1/
--- /users.py

# Defined single method endpoints (usually functional endpoints, like "/api/rest/v1/users/acquire_x")
# Usually files are sets of async functions
- /handlers/
-- /v1/
--- /users.py

# Defined functional components for postprocessing in /views/
- /functional/
-- /v1/
--- /utils.py
--- /users.py

# Defined middlewares
- /middlewares/
-- /v1/
--- /security.py
--- /session.py

# Storage for temporary files 
- /tmp/
-- /tmp_1828x812.*

# Storage for cached files 
- /cache/
-- /cache.db
-- /file_17XSDB18.*
```



###### Router model (Package preprocessing)

```python
# Router should be associated with methods named after HTTP request methods 

@user_router.get("/", summary="Get all users")
async def get_users():
    ...

@user_router.get("/{user_id}", summary="Get a user by ID")
async def get_user(user_id: int):
    ...

@user_router.post("/", summary="Create a new user")
async def create_user(user: dict):
    ...
```



###### View model (Package postprocessing)

```python
# View classes should be associated with methods named after HTTP request methods 

class Users:
  @staticmethod
  def get(*args): # arguments are retrieved data from HTTP package
    # postprocessing / application logic
  
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



###### Functional model (Package postprocessing)

```python
# Functional model should be associated with comprehendable methods (easy to understand their meanings)
# The rule of thumb: if, besides retrieving data from databases, you also jungle with data and use complex algorithms or math, it is better to put the application logic into separate (functional) models

class Users:
  @staticmethod
  def perform_action_x(*args):
    # logic
   
  def perform_action_y(*args):
    # logic
```



###### Headers (PUT, POST)

````
{
	"location": "{config.LOCATION_BASE}/api/rest/v1/users/123",
	"cache-id": "cache_identificator",
	"cache-lt": "when it will expire"
}
````



###### Specifications for Passing Arguments in Requests

#### **GET**
- Use **query parameters** for:
  - Identifiers
  - Filters, etc.

#### **POST**
- Use **query parameters** for:
  - Result non-impacting fields (e.g., `sub`)
- Include the rest in:
  - **JSON body**
  - **File uploads** and related data via `post-form`

#### **DELETE**
- Use **query parameters** for:
  - Identifier (e.g., `id`)
- Do **not** use filters or other parameters.

#### **PUT**
- Use **query parameters** for:
  - Identifier (e.g., `id`)
- Do **not** use filters or other parameters.
- Include everything else in:
  - **JSON body**
  - **File uploads** via `post-form`



###### Authorization middleware at the main API

Implements a middleware, performing preflight check if a request's method and endpoint path are allowed by the `acs` field.

If it matches criteria, it proceeds working. Otherwise, it responds with `401 Unauthorized` error.

###### Authorization module

## Access / Refresh token

* access - {"GET": ["users", "wallets", "..."], "POST": ["users"], "PUT"..., "DELETE": ...}
* key - API KEY will allow us to generate the token
* level - 0, 1, 2, 3 (0 - admin, 1 - fluter client user, 4 - fluter client _, 2 - web client, 3 - ...)

1. `v` - API version for which it was generated 
2. `iss` - API agent that issued the token
3. `iat` - When the token was issued
4. `sub` - Client id
5. `lvl` - Client Role
6. `acs` - Access rules

![Screenshot 2025-01-11 at 9.14.43 PM](/Users/bogdanyakupov/Desktop/Screenshot 2025-01-11 at 9.14.43 PM.png)

### **1. Login**  

`POST /auth/v1/login`  

#### Request  
- **JSON Body**:  
  
  ```json
  {
    "phone": "string",
  9
  }
  ```

- #### Response  

  - **JSON Response**:  

    ```json
    {
      "message": "string",
    	"verification_token": "string"
    }
    ```

  #### Description  

  - This endpoint registers a user by their phone number.  
  - A verification code is sent to the user via messaging services like Telegram or WhatsApp.  
  - The code is delivered using RabbitMQ for asynchronous messaging.  


---

### **2. Refresh Token**  

`POST /auth/v1/refresh`  

#### Request  
- **JSON Body**:  
  ```json
  {
    "refresh_token": "string"
  }
  ```

#### Response  
- **JSON Response**:  
  
  ```json
  {
    "access_token": "string"
  }
  ```

#### Description  
- This endpoint generates a new `access_token` using a valid `refresh_token`.  
- The `refresh_token` should be active and unexpired.  

---

### **3. Register**  

`POST /auth/v1/register`  

#### Request  
- **JSON Body**:  
  ```json
  {
    "phone": "string"
  }
  ```

#### Response  
- **JSON Response**:  
  ```json
  {
    "message": "string",
    "verification_token": "string"
  }
  ```

#### Description  
- This endpoint registers a user by their phone number.  
- A verification code is sent to the user via messaging services like Telegram or WhatsApp.  
- The code is delivered using RabbitMQ for asynchronous messaging.  

---

### **4. Verify Code**  

`POST /auth/v1/verify`  

#### Request  
- **JSON Body**:  
  ```json
  {
    "code": "string",
    "verification_token": "string"
  }
  ```

#### Response  
- **JSON Response**:  
  
  ```json
  {
    "message": "string",
    "access_token": "string",
    "refresh_token": "string"
  }
  ```

#### Description  
- This endpoint verifies the code sent to the user during registration.  
- On successful verification, the user is authenticated and provided with a token pair.  

---

### **Token Format**  

Both `verification_token`, `access_token` and `refresh_token` are JWTs (JSON Web Tokens)
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
