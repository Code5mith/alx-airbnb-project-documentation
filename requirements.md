# Airbnb-like Backend System Requirements

## 1. User Authentication

### Overview
Handles user registration, login, logout, and profile management with secure JWT-based authentication.

### API Endpoints
- **POST /api/auth/register**
  - Registers a new user.
- **POST /api/auth/login**
  - Authenticates a user and returns a JWT token.
- **POST /api/auth/logout**
  - Invalidates the user’s session/token.
- **GET /api/auth/profile**
  - Retrieves authenticated user’s profile (requires JWT).

### Input/Output Specifications
- **POST /api/auth/register**
  - **Request Body**:
    ```json
    {
      "email": "string",
      "password": "string",
      "name": "string",
      "role": "string" // "guest" or "host"
    }
    ```
  - **Response** (201 Created):
    ```json
    {
      "user_id": "string",
      "email": "string",
      "name": "string",
      "role": "string",
      "message": "User registered successfully"
    }
    ```
  - **Error** (400 Bad Request):
    ```json
    {"error": "Email already exists"}
    ```
- **POST /api/auth/login**
  - **Request Body**:
    ```json
    {
      "email": "string",
      "password": "string"
    }
    ```
  - **Response** (200 OK):
    ```json
    {
      "user_id": "string",
      "email": "string",
      "name": "string",
      "role": "string",
      "token": "string" // JWT
    }
    ```
  - **Error** (401 Unauthorized):
    ```json
    {"error": "Invalid credentials"}
    ```
- **POST /api/auth/logout**
  - **Request Header**: `Authorization: Bearer <JWT>`
  - **Response** (200 OK):
    ```json
    {"message": "Logged out successfully"}
    ```
- **GET /api/auth/profile**
  - **Request Header**: `Authorization: Bearer <JWT>`
  - **Response** (200 OK):
    ```json
    {
      "user_id": "string",
      "email": "string",
      "name": "string",
      "role": "string"
    }
    ```
  - **Error** (401 Unauthorized):
    ```json
    {"error": "Invalid or missing token"}
    ```

### Validation Rules
- **Email**: Valid format (e.g., `user@example.com`), unique in DB, max 255 chars.
- **Password**: Min 8 chars, 1 uppercase, 1 lowercase, 1 number, 1 special char.
- **Name**: Non-empty, max 100 chars, alphanumeric with spaces.
- **Role**: Must be `guest` or `host`.
- **JWT**: Validated for all protected endpoints; expires in 1 hour.

### Performance Criteria
- **Latency**: <200ms for 95% of requests under 1000 concurrent users.
- **Error Rate**: <0.1% for valid requests.
- **Throughput**: Handle 5000 requests/minute for login/register.
- **Security**: Encrypt passwords (bcrypt), use HTTPS, JWT with HS256.

## 2. Property Management

### Overview
Manages property listings (create, read, update, delete) for hosts, including details and images.

### API Endpoints
- **POST /api/properties**
  - Creates a new property listing (host only).
- **GET /api/properties/{id}**
  - Retrieves a property by ID.
- **PUT /api/properties/{id}**
  - Updates a property (host only).
- **DELETE /api/properties/{id}**
  - Deletes a property (host only).
- **GET /api/properties**
  - Lists properties with filters (e.g., location, price).

### Input/Output Specifications
- **POST /api/properties**
  - **Request Header**: `Authorization: Bearer <JWT>`
  - **Request Body**:
    ```json
    {
      "title": "string",
      "description": "string",
      "location": {
        "lat": "float",
        "lon": "float",
        "address": "string"
      },
      "price_per_night": "float",
      "max_guests": "integer",
      "images": ["string"] // Base64 or URLs
    }
    ```
  - **Response** (201 Created):
    ```json
    {
      "property_id": "string",
      "title": "string",
      "description": "string",
      "location": {...},
      "price_per_night": "float",
      "max_guests": "integer",
      "images": ["string"],
      "host_id": "string"
    }
    ```
  - **Error** (400 Bad Request):
    ```json
    {"error": "Missing required fields"}
    ```
- **GET /api/properties/{id}**
  - **Response** (200 OK): Same as POST response.
  - **Error** (404 Not Found):
    ```json
    {"error": "Property not found"}
    ```
- **GET /api/properties**
  - **Query Params**: `?location=string&min_price=float&max_price=float&guests=integer`
  - **Response** (200 OK):
    ```json
    {
      "properties": [{...}, {...}] // Array of property objects
    }
    ```

### Validation Rules
- **Title**: Non-empty, max 100 chars.
- **Description**: Max 1000 chars.
- **Location**: `lat` (-90 to 90), `lon` (-180 to 180), `address` non-empty.
- **Price_per_night**: Positive float, max 10000.
- **Max_guests**: Integer, 1-50.
- **Images**: Valid Base64 or URLs, max 10 images, 5MB each.
- **Host Only**: Only users with `role: host` can create/update/delete.

### Performance Criteria
- **Latency**: <300ms for GET, <500ms for POST/PUT/DELETE (95% of requests).
- **Error Rate**: <0.1% for valid requests.
- **Throughput**: Handle 2000 property queries/minute.
- **Storage**: Store images in cloud (e.g., AWS S3), metadata in SQL DB.

## 3. Booking System

### Overview
Manages booking creation, retrieval, and cancellation, ensuring availability and payment processing.

### API Endpoints
- **POST /api/bookings**
  - Creates a booking (guest only).
- **GET /api/bookings/{id}**
  - Retrieves a booking by ID.
- **DELETE /api/bookings/{id}**
  - Cancels a booking (guest or host).
- **GET /api/bookings**
  - Lists user’s bookings (guest or host).

### Input/Output Specifications
- **POST /api/bookings**
  - **Request Header**: `Authorization: Bearer <JWT>`
  - **Request Body**:
    ```json
    {
      "property_id": "string",
      "check_in": "YYYY-MM-DD",
      "check_out": "YYYY-MM-DD",
      "guests": "integer"
    }
    ```
  - **Response** (201 Created):
    ```json
    {
      "booking_id": "string",
      "property_id": "string",
      "guest_id": "string",
      "check_in": "YYYY-MM-DD",
      "check_out": "YYYY-MM-DD",
      "guests": "integer",
      "total_price": "float",
      "status": "string" // "confirmed", "pending"
    }
    ```
  - **Error** (400 Bad Request):
    ```json
    {"error": "Dates unavailable"}
    ```
- **GET /api/bookings/{id}**
  - **Request Header**: `Authorization: Bearer <JWT>`
  - **Response** (200 OK): Same as POST response.
  - **Error** (404 Not Found):
    ```json
    {"error": "Booking not found"}
    ```
- **GET /api/bookings**
  - **Query Params**: `?user_id=string&role=string`
  - **Response** (200 OK):
    ```json
    {
      "bookings": [{...}, {...}] // Array of booking objects
    }
    ```

### Validation Rules
- **Property_id**: Valid, existing property ID.
- **Check_in/out**: Valid dates, `check_in` < `check_out`, not in past.
- **Guests**: Integer, 1 to `property.max_guests`.
- **Availability**: No overlapping bookings for same property.
- **Minimum Stay**: 1 night (configurable).
- **Guest Only**: Only `role: guest` can create bookings.

### Performance Criteria
- **Latency**: <300ms for GET, <600ms for POST/DELETE (95% of requests).
- **Error Rate**: <0.2% for valid requests.
- **Throughput**: Handle 1000 booking requests/minute.
- **Concurrency**: Lock property availability during booking creation.