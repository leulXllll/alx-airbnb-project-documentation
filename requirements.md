# Airbnb Clone Backend Requirement Specifications

This document outlines detailed requirement specifications for three key backend features of the Airbnb Clone project: User Authentication, Property Management, and Booking System. Each feature includes API endpoints, input/output specifications, validation rules, performance criteria, and database interactions.

## 1. User Authentication

### Description
Enables users to register, log in, recover passwords, and manage sessions securely, ensuring only authorized users (guests, hosts, admins) access the platform.

### API Endpoints
1. **POST /auth/register**
   - **Description**: Register a new user account.
   - **Input**:
     - **Body** (JSON):
       ```json
       {
         "email": "string",
         "password": "string",
         "first_name": "string",
         "last_name": "string",
         "phone_number": "string|null",
         "role": "string" // "guest", "host", or "admin"
       }
       ```
   - **Output**:
     - **Success** (201 Created):
       ```json
       {
         "user_id": "uuid",
         "email": "string",
         "first_name": "string",
         "last_name": "string",
         "role": "string",
         "created_at": "timestamp"
       }
       ```
     - **Error** (400 Bad Request, 409 Conflict):
       ```json
       {
         "error": "Email already exists"
       }
       ```
2. **POST /auth/login**
   - **Description**: Authenticate a user and issue a session token.
   - **Input**:
     - **Body** (JSON):
       ```json
       {
         "email": "string",
         "password": "string"
       }
       ```
   - **Output**:
     - **Success** (200 OK):
       ```json
       {
         "user_id": "uuid",
         "email": "string",
         "role": "string",
         "token": "string"
       }
       ```
     - **Error** (401 Unauthorized):
       ```json
       {
         "error": "Invalid credentials"
       }
       ```
3. **POST /auth/password-recovery**
   - **Description**: Initiate password recovery by sending a reset link.
   - **Input**:
     - **Body** (JSON):
       ```json
       {
         "email": "string"
       }
       ```
   - **Output**:
     - **Success** (200 OK):
       ```json
       {
         "message": "Password reset link sent"
       }
       ```
     - **Error** (404 Not Found):
       ```json
       {
         "error": "Email not found"
       }
       ```

### Validation Rules
- **Email**: Required, valid email format, unique in `User` table.
- **Password**: Required, min 8 characters, must include uppercase, lowercase, number, special character.
- **First Name, Last Name**: Required, alphanumeric, 2–50 characters.
- **Phone Number**: Optional, valid phone format if provided.
- **Role**: Required, one of `guest`, `host`, `admin`.
- **Token**: Validated for protected endpoints, expires after 24 hours.

### Performance Criteria
- **Response Time**: < 200 ms for login/registration (95th percentile).
- **Throughput**: 1,000 concurrent requests per second.
- **Scalability**: Support 100,000 users with load balancing.
- **Security**: Passwords hashed, tokens encrypted, HTTPS enforced.

### Database Interactions
- **User Table**: Insert user data on registration, query for login, update password for recovery.

## 2. Property Management

### Description
Allows hosts to list, update, and delete property listings, enabling guests to browse and book properties.

### API Endpoints
1. **POST /properties**
   - **Description**: Create a new property listing.
   - **Input**:
     - **Headers**: Authorization: Bearer `<token>` (host).
     - **Body** (JSON):
       ```json
       {
         "name": "string",
         "description": "string",
         "location": "string",
         "price_per_night": "number"
       }
       ```
   - **Output**:
     - **Success** (201 Created):
       ```json
       {
         "property_id": "uuid",
         "host_id": "uuid",
         "name": "string",
         "description": "string",
         "location": "string",
         "price_per_night": "number",
         "created_at": "timestamp"
       }
       ```
     - **Error** (401 Unauthorized, 403 Forbidden):
       ```json
       {
         "error": "Unauthorized or not a host"
       }
       ```
2. **PUT /properties/{property_id}**
   - **Description**: Update a property’s details.
   - **Input**:
     - **Headers**: Authorization: Bearer `<token>` (host).
     - **Path**: `property_id` (UUID).
     - **Body** (JSON):
       ```json
       {
         "name": "string",
         "description": "string",
         "location": "string",
         "price_per_night": "number"
       }
       ```
   - **Output**:
     - **Success** (200 OK):
       ```json
       {
         "property_id": "uuid",
         "name": "string",
         "description": "string",
         "location": "string",
         "price_per_night": "number",
         "updated_at": "timestamp"
       }
       ```
     - **Error** (403 Forbidden, 404 Not Found):
       ```json
       {
         "error": "Not authorized or property not found"
       }
       ```
3. **DELETE /properties/{property_id}**
   - **Description**: Delete a property listing.
   - **Input**:
     - **Headers**: Authorization: Bearer `<token>` (host).
     - **Path**: `property_id` (UUID).
   - **Output**:
     - **Success** (204 No Content): No body.
     - **Error** (403 Forbidden, 404 Not Found):
       ```json
       {
         "error": "Not authorized or property not found"
       }
       ```

### Validation Rules
- **Name**: Required, 5–100 characters, alphanumeric.
- **Description**: Required, 10–1000 characters.
- **Location**: Required, 5–200 characters.
- **Price Per Night**: Required, positive decimal, max 10,000.
- **Host ID**: Must match authenticated user, `role = host`.
- **Property ID**: Valid UUID, must exist and belong to host.

### Performance Criteria
- **Response Time**: < 150 ms for create/update/delete (95th percentile).
- **Throughput**: 500 concurrent requests per second.
- **Scalability**: Handle 10,000 listings with indexing on `location`, `price_per_night`.
- **Consistency**: Atomic updates to `Property` table.

### Database Interactions
- **Property Table**: Insert on create, update fields on update, delete with cascading `Booking`, `Payment` deletion.

## 3. Booking System

### Description
Enables guests to check property availability, create bookings, and cancel bookings, ensuring valid and conflict-free reservations.

### API Endpoints
1. **GET /properties/{property_id}/availability**
   - **Description**: Check property availability for specific dates.
   - **Input**:
     - **Path**: `property_id` (UUID).
     - **Query Parameters**:
       - `start_date`: Date (YYYY-MM-DD).
       - `end_date`: Date (YYYY-MM-DD).
   - **Output**:
     - **Success** (200 OK):
       ```json
       {
         "property_id": "uuid",
         "start_date": "date",
         "end_date": "date",
         "is_available": "boolean"
       }
       ```
     - **Error** (400 Bad Request, 404 Not Found):
       ```json
       {
         "error": "Invalid dates or property not found"
       }
       ```
2. **POST /bookings**
   - **Description**: Create a new booking.
   - **Input**:
     - **Headers**: Authorization: Bearer `<token>` (guest).
     - **Body** (JSON):
       ```json
       {
         "property_id": "uuid",
         "start_date": "date",
         "end_date": "date"
       }
       ```
   - **Output**:
     - **Success** (201 Created):
       ```json
       {
         "booking_id": "uuid",
         "property_id": "uuid",
         "user_id": "uuid",
         "start_date": "date",
         "end_date": "date",
         "total_price": "number",
         "status": "string",
         "created_at": "timestamp"
       }
       ```
     - **Error** (400 Bad Request, 409 Conflict):
       ```json
       {
         "error": "Property not available or invalid dates"
       }
       ```
3. **PATCH /bookings/{booking_id}/cancel**
   - **Description**: Cancel a booking.
   - **Input**:
     - **Headers**: Authorization: Bearer `<token>` (guest).
     - **Path**: `booking_id` (UUID).
   - **Output**:
     - **Success** (200 OK):
       ```json
       {
         "booking_id": "uuid",
         "status": "canceled"
       }
       ```
     - **Error** (403 Forbidden, 404 Not Found):
       ```json
       {
         "error": "Not authorized or booking not found"
       }
       ```

### Validation Rules
- **Property ID**: Valid UUID, must exist in `Property`.
- **Start Date, End Date**: Required, valid dates, `end_date > start_date`, within 2 years.
- **Availability**: No overlapping confirmed/pending bookings.
- **Total Price**: Calculated as `price_per_night * (end_date - start_date)`, positive decimal.
- **Status**: One of `pending`, `confirmed`, `canceled`; only `pending`/`confirmed` can be canceled.
- **User ID**: Must match authenticated user, `role = guest`.

### Performance Criteria
- **Response Time**: < 100 ms for availability checks, < 200 ms for create/cancel (95th percentile).
- **Throughput**: 2,000 availability checks, 500 booking requests per second.
- **Scalability**: Support 1 million bookings with indexing on `property_id`, `start_date`, `end_date`.
- **Concurrency**: Use transactions to prevent double-booking.

### Database Interactions
- **Booking Table**: Query for availability, insert on create, update `status` on cancel.
- **Property Table**: Query `price_per_night` for `total_price`.