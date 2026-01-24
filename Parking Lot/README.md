# Parking Lot System Design

## 1. Problem Statement
Design a parking lot system that allows vehicles to enter, park, exit,
and pay parking fees based on the duration of stay.

## 2. Assumptions
- Parking lot has fixed capacity
- Supports multiple floors
- Supports limited vehicle types (Bike, Car, Truck)
- One vehicle occupies one parking spot
- Pricing is time-based
- Payment can be online or manual
- Parking lot works 24×7
- No advance booking in the initial version
- System is designed for a single parking location

## 3. High-Level Architecture
Vehicle → Entry Gate → Backend → Database  
Vehicle → Exit Gate → Backend → Payment → Exit

## 4. Relevant Tech Stack
Frontend: React (Admin Dashboard)  
Backend: Java Spring Boot  
Database: MySQL  
Cache: Redis  
Authentication: JWT  
Payment: Razorpay  
Deployment: AWS EC2  

## 5. Key Components

Entry Gate  
- Detects vehicle entry  
- Calls backend to allocate spot  
- Generates parking ticket  

Exit Gate  
- Accepts ticket  
- Calculates fee  
- Confirms payment  
- Releases spot  

Parking Spot Manager  
- Tracks available spots  
- Assigns nearest suitable spot  

Pricing Engine  
- Calculates fee based on time difference  

## 6. Database Design

ParkingSpot  
- spot_id (INT)  
- floor_no (INT)  
- spot_type (ENUM: BIKE, CAR, TRUCK)  
- is_available (BOOLEAN)  

ParkingTicket  
- ticket_id (INT)  
- vehicle_number (VARCHAR)  
- entry_time (TIMESTAMP)  
- exit_time (TIMESTAMP)  
- spot_id (INT)  
- amount (DECIMAL)  

## 7. API Design
POST /entry → Create parking ticket  
POST /exit/{ticketId} → Exit & payment  
GET /spots/available → View available spots  
GET /admin/reports → Daily usage report  

## 8. Parking Fee Logic
Assumption:  
First 1 hour → ₹30  
Each extra hour → ₹20  

totalFee = baseFee + (extraHours × rate)

## 9. Non-Functional Requirements

Scalability  
- Backend can be scaled horizontally  
- Redis reduces DB load  

Availability  
- DB replication (read replicas)  
- Graceful failure handling  

Security  
- JWT-based admin authentication  
- Secure payment gateway integration  

Performance  
- Cache frequently accessed data  
- Index on spot_type and is_available  

## 10. Failure Scenarios & Handling
DB down → Stop new entries  
Payment failure → Manual override  
Spot mismatch → Transaction rollback  

## 11. Trade-Offs
- Monolith instead of microservices → simpler to manage  
- MySQL instead of NoSQL → structured relationships  
- Redis optional → added only for performance  

## 12. Future Enhancements
- Mobile app for users  
- Vehicle number plate recognition  
- Dynamic pricing  
- Pre-booking slots  
- Multi-location support
