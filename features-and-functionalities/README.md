# Airbnb Clone – Backend Requirements

## Introduction
Backend development involves creating the server-side logic, database management, and API integrations that power a web application.  
This document outlines the backend requirements for the **Airbnb Clone Project**, focusing on the **core functionalities**, **technical requirements**, and **non-functional requirements** needed to make the app functional, user-friendly, and efficient.  

---

## Core Functionalities

### 1. User Management
- **User Registration**
  - Allow users to sign up as **guests** or **hosts**  
  - Secure authentication using **JWT (JSON Web Tokens)**
- **User Login & Authentication**
  - Login with **email + password**  
  - Support for **OAuth** (Google, Facebook)
- **Profile Management**
  - Update profile details: **photo, contact info, preferences**

---

### 2. Property Listings Management
- **Add Listings**
  - Hosts can create property listings with:  
    - Title  
    - Description  
    - Location  
    - Price  
    - Amenities  
    - Availability  
- **Edit/Delete Listings**
  - Hosts can update or remove their property listings  

---

### 3. Search & Filtering
- Search properties by:  
  - Location  
  - Price range  
  - Number of guests  
  - Amenities (Wi-Fi, pool, pet-friendly, etc.)  
- Include **pagination** for large datasets  

---

### 4. Booking Management
- **Booking Creation**
  - Guests can book properties for specific dates  
  - Validate dates to prevent double bookings  
- **Booking Cancellation**
  - Guests or hosts can cancel bookings based on cancellation policy  
- **Booking Status**
  - Track booking statuses: **pending, confirmed, canceled, completed**  

---

### 5. Payment Integration
- Secure payment gateways: **Stripe, PayPal**  
- **Upfront payments** by guests  
- **Automatic payouts** to hosts after booking completion  
- Support for **multiple currencies**  

---

### 6. Reviews & Ratings
- Guests can leave reviews and ratings for properties  
- Hosts can respond to reviews  
- Reviews must be linked to **specific bookings** to prevent abuse  

---

### 7. Notifications System
- **Email and in-app notifications** for:  
  - Booking confirmations  
  - Cancellations  
  - Payment updates  

---

### 8. Admin Dashboard
- Admin interface for managing:  
  - Users  
  - Listings  
  - Bookings  
  - Payments  
