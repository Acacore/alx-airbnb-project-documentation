# Airbnb Clone Use Case Diagram Documentation

## Overview

This document describes the **Use Case Diagram** for the Airbnb Clone project, a rental marketplace application. The diagram illustrates the interactions between users (actors) and the application's core functionalities, as specified in the project requirements. The use case diagram provides a high-level view of the system’s behavior from the perspective of Guests, Hosts, Admins, and the System itself.

The diagram is defined using **PlantUML**, a text-based tool for generating UML diagrams, and can be rendered in tools like Draw.io, PlantUML online, or Kroki. This README includes instructions for rendering the diagram, exporting it as a PNG, and adjusting the layout to ensure a vertical orientation, as requested.

## Purpose

The use case diagram serves the following purposes:
- **Visualize User Interactions**: Captures how actors (Guest, Host, Admin, System) interact with the system’s core functionalities, such as registering, booking properties, and managing listings.
- **Clarify Functional Scope**: Maps the **Core Functionalities** section of the project requirements to specific use cases, ensuring all key interactions are represented.
- **Support Development**: Provides a reference for developers and stakeholders to understand the system’s behavior and guide implementation.

## Actors

The following actors interact with the system:
- **Guest**: Searches for, books, and reviews properties.
- **Host**: Creates, manages, and responds to reviews for property listings.
- **Admin**: Manages users, listings, bookings, and payments via an admin dashboard.
- **System**: Performs automated tasks, such as sending notifications or processing payouts.

## Use Cases

The use cases are grouped into subsystems based on the **Core Functionalities** section of the project requirements. Each subsystem contains related actions that users or the system can perform.

### User Management
- Register as Guest (Guest)
- Register as Host (Host)
- Log In (Guest, Host, Admin)
- Authenticate via OAuth (Guest, Host, extends Register and Log In)
- Manage Profile (Guest, Host)

### Property Listings Management
- Create Property Listing (Host)
- Edit Property Listing (Host)
- Delete Property Listing (Host)

### Search and Filtering
- Search Properties (Guest)
- Filter Properties (Guest, extends Search Properties)

### Booking Management
- Create Booking (Guest, includes Make Payment)
- Cancel Booking (Guest, Host)
- View Booking Status (Guest, Host)

### Payment Integration
- Make Payment (Guest)
- Receive Payout (Host, System)

### Reviews and Ratings
- Submit Review and Rating (Guest)
- Respond to Review (Host)

### Notifications System
- Receive Notification (Guest Host)
- Send Notification (System)

### Admin Dashboard
- Manage Users (Admin)
- Manage Listings (Admin)
- Manage Bookings (Admin)
- Manage Payments (Admin)

## Relationships
- **Include**: Indicates a mandatory step (e.g., `Create Booking` includes `Make Payment`).
- **Extend**: Indicates an optional action (e.g., `Search Properties` extends to `Filter Properties`).
- **Direct Interaction**: Actors are connected to use cases with solid lines.

## PlantUML Code

The use case diagram is defined in PlantUML, optimized for a **vertical layout** to address the requirement for a taller, narrower diagram. The code includes directives like `top to bottom direction` and `skinparam` settings to encourage vertical stacking of actors and subsystems.

```plantuml
@startuml Airbnb Clone Use Case Diagram
!theme clean
top to bottom direction
skinparam ranksep 100
skinparam nodesep 50
skinparam monochrome true

actor Guest
actor Host
actor Admin
actor System

package "Airbnb Clone System" {
  package "User Management" {
    (Register as Guest) .> (Authenticate via OAuth) : <<extend>>
    (Register as Host) .> (Authenticate via OAuth) : <<extend>>
    (Log In) .> (Authenticate via OAuth) : <<extend>>
    Guest --> (Register as Guest)
    Guest --> (Log In)
    Guest --> (Manage Profile)
    Host --> (Register as Host)
    Host --> (Log In)
    Host --> (Manage Profile)
    Admin --> (Log In)
  }

  package "Property Listings Management" {
    Host --> (Create Property Listing)
    Host --> (Edit Property Listing)
    Host --> (Delete Property Listing)
  }

  package "Search and Filtering" {
    (Search Properties) .> (Filter Properties) : <<extend>>
    Guest --> (Search Properties)
  }

  package "Booking Management" {
    (Create Booking) .> (Make Payment) : <<include>>
    Guest --> (Create Booking)
    Guest --> (Cancel Booking)
    Guest --> (View Booking Status)
    Host --> (Cancel Booking)
    Host --> (View Booking Status)
  }

  package "Payment Integration" {
    Guest --> (Make Payment)
    Host --> (Receive Payout)
    System --> (Receive Payout)
  }

  package "Reviews and Ratings" {
    Guest --> (Submit Review and Rating)
    Host --> (Respond to Review)
  }

  package "Notifications System" {
    Guest --> (Receive Notification)
    Host --> (Receive Notification)
    System --> (Send Notification)
  }

  package "Admin Dashboard" {
    Admin --> (Manage Users)
    Admin --> (Manage Listings)
    Admin --> (Manage Bookings)
    Admin --> (Manage Payments)
  }
}

' Position actors vertically
Guest -[hidden]d- Host
Host -[hidden]d- Admin
Admin -[hidden]d- System

@enduml