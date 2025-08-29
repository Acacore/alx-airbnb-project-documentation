# Backend Requirement Specifications

**Project:** Airbnb Clone (Property Rental Platform)  
**File:** `requirements.md`  
**Prepared:** August 29, 2025

> **Note:**  
> 1. This document is the authoritative requirements specification for the backend features described below.  
> 2. All example request/response payloads are provided as JSON code blocks for clarity and copy/paste use.  
> 3. Implementation notes (transactions, locks, indexing) are included to help developers meet validation and performance criteria.

---

## Table of Contents
1. [Scope & Assumptions](#scope--assumptions)  
2. [Feature 1 — User Authentication](#feature-1---user-authentication)  
3. [Feature 2 — Property Management](#feature-2---property-management)  
4. [Feature 3 — Booking System](#feature-3---booking-system)  
5. [Additional Features (summaries)](#additional-features-summaries)  
6. [Data Stores & Indexing (appendix)](#data-stores--indexing-appendix)  
7. [Non-Functional Requirements](#non-functional-requirements)  
8. [Change Log](#change-log)

---

## Scope & Assumptions
- Backend is RESTful by default (JSON over HTTP). GraphQL may be used for complex list/detail queries (optional).  
- Primary database: **PostgreSQL** (ACID transactions). Consider read replicas for scaling.  
- Authentication: **JWT** for stateless auth; refresh tokens optional. OAuth via Google/Facebook supported.  
- File storage (images): **S3 / Cloudinary** recommended.  
- Payment gateway integration: **Stripe** and/or **PayPal**. Webhooks must be validated.  
- All timestamps use ISO 8601 (UTC).

---

## Feature 1 — User Authentication

### Objective
Securely register, authenticate, and manage user profiles for `guest`, `host`, and `admin` roles.

### Actors
- Guest, Host, Admin, OAuth Provider

### Endpoints

| Endpoint                        | Method | Auth Required | Description |
|--------------------------------|--------|---------------|-------------|
| `/api/auth/signup`             | POST   | No            | Register user (guest|host) |
| `/api/auth/login`              | POST   | No            | Authenticate & return JWT |
| `/api/auth/logout`             | POST   | Yes           | Revoke token / logout |
| `/api/auth/refresh`            | POST   | Yes (refresh token) | Refresh access token |
| `/api/auth/me`                 | GET    | Yes           | Get current user profile |
| `/api/auth/password-reset`     | POST   | No            | Request reset email |
| `/api/auth/password-reset/confirm` | PUT | No            | Confirm new password |
| `/api/auth/oauth/callback`     | POST   | No            | OAuth callback (Google/Facebook) |

### Example: Signup (request / response)
```json
POST /api/auth/signup
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "StrongPass!23",
  "role": "guest"   // "guest" or "host"
}
json
Copy code
201 Created
{
  "user_id": "u_12345",
  "email": "john@example.com",
  "role": "guest",
  "message": "User registered successfully"
}
Example: Login
json
Copy code
POST /api/auth/login
{
  "email": "john@example.com",
  "password": "StrongPass!23"
}
json
Copy code
200 OK
{
  "access_token": "eyJhbGciOi...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "refresh_token": "rfrsh_abc123"
}
Error example
json
Copy code
401 Unauthorized
{
  "error": "Invalid credentials"
}
Validation Rules
email: required, valid format, unique.

password: min 8 characters; must include uppercase, lowercase, number, special character.

username: 3–20 chars, alphanumeric + underscores.

role: must be guest or host.

Rate-limit signup/login endpoints to mitigate brute-force attacks.

Security Considerations
Store password hashes using bcrypt/argon2.

Access tokens short-lived (e.g., 1 hour); rotate via refresh tokens stored securely.

Use HTTPS only.

Implement account lockout policies after repeated failed attempts.

OAuth tokens validated and user info normalized.

Performance Criteria
Auth endpoints latency: ≤ 300ms under normal load.

Support 1,000 concurrent login requests (scale horizontally).

JWT validation must be < 5ms per request in hot path.

Feature 2 — Property Management
Objective
Enable hosts to create, update, list, and delete property listings; allow guests to search and view listings.

Actors
Host, Guest, Admin

Endpoints
Endpoint	Method	Auth Required	Description
/api/properties	POST	Yes (host)	Create a new property
/api/properties	GET	No	List/search properties (filters & pagination)
/api/properties/{id}	GET	No	Retrieve single property
/api/properties/{id}	PUT	Yes (host)	Update property (owner only)
/api/properties/{id}	DELETE	Yes (host/admin)	Delete property
/api/properties/{id}/images	POST	Yes (host)	Upload images (multipart)

Create Property (example)
json
Copy code
POST /api/properties
{
  "title": "Modern 2BR Apartment",
  "description": "2-bedroom apartment with ocean view, free WiFi.",
  "price_per_night": 95.00,
  "currency": "USD",
  "location": {
    "address": "12 Ocean Drive",
    "city": "Accra",
    "country": "Ghana",
    "lat": 5.6037,
    "lng": -0.1870
  },
  "amenities": ["wifi","kitchen","air_conditioning"],
  "max_guests": 4,
  "available_from": "2025-09-01",
  "available_to": "2026-09-01"
}
json
Copy code
201 Created
{
  "property_id": "p_67890",
  "message": "Property created successfully"
}
Search / List (example)
GET /api/properties?city=Accra&min_price=50&max_price=150&guests=2&page=1&limit=20

Response (truncated):

json
Copy code
200 OK
{
  "page": 1,
  "limit": 20,
  "total": 345,
  "results": [
    {
      "property_id": "p_67890",
      "title": "Modern 2BR Apartment",
      "price_per_night": 95,
      "currency": "USD",
      "location": { "city": "Accra", "country": "Ghana" },
      "thumbnail_url": "https://cdn.example.com/..."
    }
  ]
}
Validation Rules
title: required, 5–100 chars.

description: max 1000 chars.

price_per_night: numeric, > 0.

max_guests: integer ≥ 1.

location.city and location.country: required.

Image uploads: limit size (e.g., 5 MB each), validate file type, store in S3/Cloudinary.

Authorization
Only the owner (host) or admin may modify/delete a property. Enforce RBAC checks on all protected endpoints.

Performance Criteria
Property creation/update latency: ≤ 500ms.

Search queries for common filters: ≤ 400ms (use caching / indexed queries).

System should smoothly support 100,000 property records with proper DB indexing (see appendix).

Implementation Notes
Index location.city, price_per_night, max_guests, and availability in DB.

Use a full-text search or ElasticSearch for complex searching if needed.

Cache hot search results in Redis (TTL as appropriate).

Feature 3 — Booking System
Objective
Allow guests to request and confirm bookings; prevent double-booking; support cancellations and status transitions.

Actors
Guest, Host, Payment Gateway, Admin

Endpoints
Endpoint	Method	Auth Required	Description
/api/bookings	POST	Yes (guest)	Create booking request
/api/bookings/{id}	GET	Yes	Get booking details (owner/guest/admin)
/api/bookings/{id}/status	PUT	Yes (host/admin)	Update booking status (approve/reject/complete)
/api/bookings/{id}/cancel	POST	Yes (guest/host)	Cancel booking (apply policy)
/api/bookings/user/{user_id}	GET	Yes	Get a user's bookings (guest/host)

Create Booking (example)
json
Copy code
POST /api/bookings
{
  "property_id": "p_67890",
  "guest_id": "u_12345",
  "check_in": "2025-09-10",
  "check_out": "2025-09-15",
  "guests": 2,
  "payment_method": "stripe"
}
json
Copy code
201 Created
{
  "booking_id": "b_9001",
  "property_id": "p_67890",
  "guest_id": "u_12345",
  "status": "pending",
  "total_price": 475.00,
  "currency": "USD",
  "message": "Booking request submitted"
}
Booking Flow & Double-Booking Prevention
Validate input (dates, guest count, auth).

Use a DB transaction with a SELECT ... FOR UPDATE (or application-level distributed lock) on the property availability / calendar rows to check overlapping bookings.

If available, create booking row with status = pending and reserve the dates (atomic).

Trigger payment initiation (if payment required immediately). If payment fails, rollback reservation.

Upon successful payment, set status = confirmed. Notify host/guest.

Validation Rules
check_in and check_out must be valid ISO dates; check_out > check_in.

check_in cannot be in the past.

Number of guests ≤ property max_guests.

Guest cannot book their own property.

Prevent overlapping bookings for a property using transactional checks.

Status Model
pending → confirmed → completed

pending/confirmed → canceled

rejected (by host)

Track timestamps for each status transition.

Cancellation / Refunds
Cancellation endpoint enforces policy. Example: full refund if canceled > 48 hours prior; partial or no refund otherwise. Exact rules configurable per property/host.

Payment Hook Integration (example)
/api/webhooks/payments/stripe (POST) — accept verified Stripe webhook for successful payment → update booking status.

Error example: Overlap
json
Copy code
409 Conflict
{
  "error": "Property is already booked for selected dates",
  "conflicting_booking_id": "b_8899"
}
Performance Criteria
Booking availability check: ≤ 300–700ms (depending on load).

Booking creation end-to-end (including payment initiation): ≤ 800ms.

Support 5,000 concurrent booking operations with horizontal scaling and database optimization.

Additional Features (summaries)
Payments
Endpoints: /api/payments/checkout (POST), /api/webhooks/payments/stripe (POST).

Required: PCI compliance best practices or use hosted checkout flows.

Performance: payment authorizations typically < 2s (depends on gateway).

Reviews & Ratings
Endpoints: /api/reviews (POST), /api/properties/{id}/reviews (GET).

Reviews tied to booking_id to prevent fraudulent reviews.

Notifications
Use SendGrid / Mailgun for emails; push/in-app via message queue (RabbitMQ / Redis Streams).

Endpoints: /api/notifications (internal).

Events: booking_confirmed, booking_canceled, payment_completed.

Admin Dashboard
Endpoints for admins to manage users, listings, payments, and bookings.

Actions logged for audit trail.

Data Stores & Indexing (Appendix)
Core Tables (minimal column examples)
users (id, email, password_hash, role, name, created_at)

properties (id, host_id, title, price_per_night, location_city, max_guests, created_at)

bookings (id, property_id, guest_id, check_in, check_out, status, total_price)

payments (id, booking_id, amount, currency, gateway, status, gateway_txn_id)

reviews (id, booking_id, property_id, guest_id, rating, comment)

Recommended Indexes
properties(location_city), properties(price_per_night), properties(max_guests)

bookings(property_id, check_in, check_out) (for overlap checks)

users(email) unique

Caching
Use Redis for caching search results, session management (if used), rate limiting.

Non-Functional Requirements
Scalability
Microservice / modular architecture recommended.

Use load balancers, autoscaling groups, and read replicas.

Security
HTTPS required.

Secrets stored in vaults.

Input validation and sanitization (protect against SQL injection).

Rate limit sensitive endpoints.

Performance
99.9% uptime SLA for core services.

Response time targets documented per endpoint above.

Observability
Centralized logging (ELK / EFK), metrics (Prometheus + Grafana), and tracing (OpenTelemetry).

Alerting on error rates, latency, and resource saturation.

Testing
Unit + integration tests; API contract tests.

CI pipeline to run test suite and deploy.