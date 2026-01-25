# Car Rental System Design

## 1. Problem Statement
Design a car rental system that allows users to search cars, book rentals,
pick up and return cars, and calculate rental charges while preventing
double booking of the same car.

## 2. Assumptions
- System operates in a single city
- Fixed number of cars
- Each car belongs to one category
- One car can be rented by only one user at a time
- Pricing is time-based (per day)
- No surge pricing in the initial version
- Self-drive only (no driver service)
- Users must be registered
- Payment is done before pickup
- Single timezone

## 3. Key Requirements

Functional Requirements
- User registration and login
- Search available cars by date and type
- Book a car
- Cancel booking
- Pick up and return car
- Calculate rental cost

Non-Functional Requirements
- Consistency (no double booking)
- Concurrency handling
- Low latency
- Easy scalability

## 4. High-Level Architecture
User → Web / Mobile App → Backend Service → Database  
(Optional) Payment Gateway → Confirmation

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Payment: Razorpay  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- name
- email
- licenseNumber

Car
- carId
- model
- category (SUV, Sedan, Hatchback)
- status (AVAILABLE, BOOKED, MAINTENANCE)

Booking
- bookingId
- userId
- carId
- startDate
- endDate
- totalAmount
- status

## 7. Database Design (Short)

Car
- car_id (INT)
- model (VARCHAR)
- category (ENUM)
- status (ENUM)

Booking
- booking_id (INT)
- user_id (INT)
- car_id (INT)
- start_date (DATE)
- end_date (DATE)
- status (ENUM)
- amount (DECIMAL)

Index: (car_id, start_date, end_date)

## 8. Preventing Double Booking

Overlap Rule:
(start1 < end2) AND (start2 < end1)

Booking Flow:
- Start database transaction
- Check overlapping bookings for selected car
- Reject if conflict exists
- Create booking and mark car as BOOKED
- Commit transaction

## 9. Handling Concurrent Booking Requests

Problem:
- Multiple users attempt to book the same car for the same dates

Solution:
- Database transaction with row-level locking

SELECT * FROM booking
WHERE car_id = ?
AND start_date < :end
AND end_date > :start
FOR UPDATE;

This ensures only one booking succeeds.

## 10. Pricing Logic
Assumptions:
- Sedan → ₹2000 per day
- SUV → ₹3000 per day

totalPrice = numberOfDays × dailyRate

## 11. API Design
GET /cars/available  
POST /bookings  
DELETE /bookings/{id}  
GET /bookings/user/{id}  
POST /return/{bookingId}  

## 12. Failure Handling
- Payment failure → booking cancelled
- Concurrent conflict → retry or reject
- Database down → disable booking

## 13. Scalability Considerations
- Cache availability by date range
- Scale backend horizontally
- Partition bookings by date
- Async email notifications

## 14. Trade-Offs
- Relational database for strong consistency
- Monolithic architecture for simplicity
- Redis for performance improvement
- Avoid microservices to reduce complexity

## 15. Future Enhancements
- Multi-city support
- Hour-based rentals
- Damage and insurance module
- Driver option
- Dynamic pricing
- Mobile application
