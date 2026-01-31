# 🎬 Movie Booking System

A comprehensive online movie ticket booking platform that caters to both **B2B (theatre partners)** and **B2C (end customers)** clients.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features Implemented](#features-implemented)
- [Technology Stack](#technology-stack)
- [Architecture](#architecture)
- [Design Patterns](#design-patterns)
- [API Documentation](#api-documentation)
- [Getting Started](#getting-started)
- [Design Diagrams](#design-diagrams)
- [Key Design Decisions](#key-design-decisions)
- [Testing](#testing)

---

## 🎯 Overview

This platform enables:
- **Theatre Partners**: Onboard theatres, manage shows, and access a larger customer base
- **End Customers**: Browse movies across cities, languages, and genres, and book tickets seamlessly

---

## ✨ Features Implemented

### 📖 Read Scenarios
- ✅ **Browse Shows**: View theatres running selected movies in a city with show timings by date
- ✅ **Offers Management**: View available offers in selected cities and theatres
  - 50% discount on the 3rd ticket
  - 20% discount on afternoon shows

### ✍️ Write Scenarios
- ✅ **Ticket Booking**: Book movie tickets by selecting theatre, timing, and preferred seats
- ✅ **Show Management**: Theatres can create, update, and delete shows
- ✅ **Bulk Operations**: Support for bulk booking and cancellation
- ✅ **Seat Inventory**: Theatres can allocate and update seat inventory for shows

---

## 🛠️ Technology Stack

| Component | Technology |
|-----------|-----------|
| **Language** | Java 21 |
| **Framework** | Spring Boot 4.0.2 |
| **Build Tool** | Maven |
| **API Documentation** | SpringDoc OpenAPI 3.0.1 (Swagger UI) |
| **Utilities** | Lombok |
| **Testing** | JUnit 5, Spring Boot Test |
| **Data Storage** | In-Memory (DummyDataStore) - Easily replaceable with JPA/Hibernate |

---

## 🏗️ Architecture

### Layered Architecture

```
┌─────────────────────────────────────┐
│     Controllers (REST APIs)         │  ← Presentation Layer
├─────────────────────────────────────┤
│     Services (Business Logic)       │  ← Service Layer
├─────────────────────────────────────┤
│     Repositories (Data Access)      │  ← Data Access Layer
├─────────────────────────────────────┤
│     Models (Domain Entities)        │  ← Domain Layer
└─────────────────────────────────────┘
```

### Key Components

#### 1. **Controllers**
- `ShowController` - Show browsing and booking operations
- `TheatreController` - Theatre and show management
- `OfferController` - Offer retrieval

#### 2. **Services**
- `ShowService` - Core booking logic with thread-safe seat locking
- `TheatreService` - Theatre and show management
- `OfferService` - Offer management
- `PricingService` - Dynamic pricing with offer calculations

#### 3. **Repositories**
- `ShowRepository` - Show data access
- `TheatreRepository` - Theatre data access
- `BookingRepository` - Booking data access
- `OfferRepository` - Offer data access

#### 4. **Domain Models**
- `Theatre` → `Screen` → `Show` → `Seat` (Hierarchical composition)
- `Movie` - Film details
- `Booking` - Customer reservations
- `Offer` - Discount and promotion details

---

## 🎨 Design Patterns

| Pattern | Implementation | Purpose |
|---------|---------------|---------|
| **Layered Architecture** | Controllers → Services → Repositories | Separation of concerns |
| **Repository Pattern** | Data access abstraction | Easy database switching |
| **Strategy Pattern** | `PricingService` interface | Flexible pricing strategies |
| **DTO Pattern** | Request/Response objects | API contract separation |
| **Singleton Pattern** | `DummyDataStore` | Centralized in-memory storage |
| **Builder Pattern** | Lombok `@Builder` | Clean object creation |

---

## 📡 API Documentation

### Base URL
```
http://localhost:8080
```

### Swagger UI
```
http://localhost:8080/swagger-ui.html
```

### Key Endpoints

#### Theatre Management (B2B)
```http
POST   /api/v1/theatres                           # Onboard theatre
POST   /api/v1/theatres/{theatreId}/screen/{screenId}  # Create show
PUT    /api/v1/theatres/show/{showId}             # Update show
DELETE /api/v1/theatres/show/{showId}             # Delete show
```

#### Show Browsing & Booking (B2C)
```http
GET    /api/v1/shows?movie={movie}&city={city}&date={date}  # Browse shows
POST   /api/v1/shows/{showId}/book                # Book tickets
DELETE /api/v1/shows/booking/{bookingId}          # Cancel booking
POST   /api/v1/shows/{showId}/bulk-book           # Bulk booking
DELETE /api/v1/shows/bulk-cancel                  # Bulk cancellation
```

#### Offers
```http
GET    /api/v1/offers?city={city}&theatreId={theatreId}  # Get offers
```

---

## 🚀 Getting Started

### Prerequisites
- Java 21 or higher
- Maven 3.6+

### Installation & Running

1. **Clone the repository**
```bash
cd MovieBookingSystem
```

2. **Build the project**
```bash
mvn clean install
```

3. **Run the application**
```bash
mvn spring-boot:run
```

4. **Access the application**
- API Base: `http://localhost:8080`
- Swagger UI: `http://localhost:8080/swagger-ui.html`

### Running Tests
```bash
mvn test
```

---

## 📊 Design Diagrams

### Available Diagrams

1.**ClassDiagram.png** (550 KB)
   - Complete Low-Level Design (LLD)
   - All classes with methods and attributes
   - Comprehensive relationships and dependencies

2.**SequenceDiagram.png** 
   - Sequence diagram for booking flow
   - Shows interaction between components

### Domain Model Hierarchy
```
Theatre
  └── Screen
       └── Show
            └── Seat

Movie ──> Show (one-to-many)
Booking ──> Show (references)
Theatre ──> Offer (has)
```

---

## 🔑 Key Design Decisions

### 1. **Thread-Safe Booking**
- **Problem**: Prevent double booking in concurrent scenarios
- **Solution**: `ReentrantLock` per seat with timeout
- **Implementation**: `lockForSeat()` method in `ShowService`

### 2. **Seat State Management**
```
AVAILABLE → BLOCKED → BOOKED
     ↑          ↓
     └──────────┘ (on payment failure/timeout)
```

### 3. **Payment Hold Mechanism**
- Seats are **BLOCKED** during payment processing
- Automatic release after timeout (configurable)
- Prevents inventory loss while ensuring availability

### 4. **Dynamic Pricing**
- Interface-based design (`PricingService`)
- Easy to add new pricing strategies
- Current implementation: `DefaultPricingService`
- Supports multiple offer types:
  - Percentage discounts
  - Conditional offers (3rd ticket, afternoon shows)

### 5. **Extensibility**
- Repository pattern allows easy database integration
- Service interfaces enable strategy pattern
- DTOs separate API contracts from domain models
- Modular design supports microservices migration

---

## 🧪 Testing

### Test Coverage
- ✅ Unit tests for services
- ✅ Integration tests for controllers
- ✅ Concurrency tests for booking scenarios

---

## 📈 Non-Functional Considerations

### Scalability
- Stateless service design
- Ready for horizontal scaling
- Database connection pooling (when integrated)
- Caching strategy for frequently accessed data

### Performance
- In-memory operations for fast response
- Optimized locking mechanism
- Bulk operations support

### Security
- Input validation on all endpoints
- Exception handling with proper error messages

### Monitoring
- Structured logging
- Exception tracking
