# Cab Booking Platform – System Design

## 1. Problem Statement
Design a cab booking platform that allows users to request rides, matches
them with nearby drivers, tracks trips in real time, and completes
payments reliably while handling heavy concurrency and low-latency
requirements.

## 2. Assumptions
- Single city in the first version
- Only point-to-point rides
- Fixed pricing model (no surge pricing)
- Online payments only
- One active ride per user and per driver
- GPS updates every few seconds
- Single timezone

## 3. Key Requirements

Functional Requirements
- User login
- Request a cab
- Match with nearby driver
- Live ride tracking
- Trip completion
- Payment processing

Non-Functional Requirements
- Low latency driver matching
- High availability
- Consistency (no double booking)
- Scalability during peak hours

## 4. High-Level Architecture
User App → Ride Service → Matching Service → Driver App  
↓  
Location / Maps Service  
↓  
Payment Service  

## 5. Relevant Tech Stack
Frontend: React / Mobile App  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache & Geo Search: Redis with GEO indexing  
Messaging: Kafka  
Maps: Google Maps API  
Realtime Updates: WebSocket  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- name
- rating

Driver
- driverId
- currentLocation
- availability

Ride
- rideId
- userId
- driverId
- status (REQUESTED / ONGOING / COMPLETED)

Location Service
- Stores live GPS updates from drivers

## 7. Database Design (Simplified)

Ride
- ride_id (BIGINT)
- user_id (BIGINT)
- driver_id (BIGINT)
- status (ENUM)
- start_time (TIMESTAMP)
- end_time (TIMESTAMP)

Driver
- driver_id (BIGINT)
- status (ENUM)
- lat (DOUBLE)
- lng (DOUBLE)

Indexes:
- ride(status)
- geo-location on driver coordinates

## 8. Cab Request and Matching Flow

1. User requests a ride with pickup and drop location
2. Ride Service creates a ride with status REQUESTED
3. Matching Service:
   - Finds nearby available drivers using Redis GEO
   - Sorts drivers by distance
   - Sends request to the nearest driver
4. Driver accepts the ride
5. Ride status changes to ONGOING

This ensures one driver is assigned to one ride.

## 9. Preventing Double Booking

Problem:
- Two users may try to book the same driver at the same time

Solution:
- Use atomic driver locking in Redis

SETNX driver:{driverId}:lock rideId EX 30

- If lock exists → driver is unavailable
- If accepted → assignment is persisted in the database

This approach is fast, thread-safe, and scalable.

## 10. Live Location Tracking
- Driver app sends GPS updates every 2–5 seconds
- Location stored in Redis
- Updates pushed to user via WebSocket

This provides low-latency real-time tracking.

## 11. Trip Completion and Payment
1. Driver ends the ride
2. Fare calculated using distance and base fare
3. Payment service charges the user
4. Ride marked as COMPLETED
5. Driver marked as AVAILABLE again

## 12. API Design
POST /rides/request        → Request cab  
GET  /rides/{id}           → Get ride status  
POST /drivers/location     → Update driver GPS  
POST /rides/complete       → Complete ride  
POST /payment/pay          → Make payment  

## 13. Failure Handling
- No drivers available → notify user and retry
- Driver cancels → re-run matching
- Payment failure → retry or fallback
- Redis down → fallback to database
- Network failure → retry with idempotency

## 14. Scalability Considerations
- Redis GEO for fast nearby driver lookup
- Kafka for async ride events
- Stateless backend services
- Partition rides by city

## 15. Trade-Offs
- Redis is fast but in-memory
- PostgreSQL provides reliable trip history
- WebSocket enables real-time updates but is stateful
- Monolith initially for simplicity and debugging

## 16. Future Enhancements
- Surge pricing
- Driver ratings
- Ride pooling
- Scheduled rides
- Multi-city support
- Fraud detection
