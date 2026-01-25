# Movie Ticket Booking System Design

## 1. Problem Statement
Design an online movie ticket booking system that allows users to browse
movies, view shows, select seats, book tickets, and make payments while
preventing double booking of seats under high concurrency.

## 2. Assumptions
- System operates in multiple cities
- Each city has multiple theaters
- Each theater has multiple screens
- Each screen has fixed seats
- A seat can be booked by only one user per show
- Seat selection is mandatory
- Booking timeout if payment is not completed
- Same timezone per city
- No dynamic pricing in initial version
- Online payments only

## 3. Key Requirements

Functional Requirements
- Browse movies by city
- View theaters and show timings
- Select seats
- Book tickets
- Make payment
- Receive booking confirmation

Non-Functional Requirements
- Strong consistency
- High concurrency handling
- Low latency for seat availability
- Scalability during peak hours

## 4. High-Level Architecture
User → Web/Mobile App → Booking Service → Seat Locking  
→ Payment → Confirmation  
(Cache and Database used internally)

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Payment: Razorpay  
Messaging: Kafka (optional)  
Deployment: AWS EC2  

## 6. Core Components

Movie
- movieId
- title
- language
- duration

Theater
- theaterId
- city
- name

Screen
- screenId
- theaterId
- totalSeats

Show
- showId
- movieId
- screenId
- startTime

Seat
- seatId
- row
- number

Booking
- bookingId
- userId
- showId
- status (PENDING / CONFIRMED / CANCELLED)

## 7. Database Design (Short)

Show
- show_id (INT)
- movie_id (INT)
- screen_id (INT)
- start_time (TIMESTAMP)

Seat
- seat_id (INT)
- screen_id (INT)

SeatBooking
- booking_id (INT)
- show_id (INT)
- seat_id (INT)
- status (ENUM)

Unique Constraint: (show_id, seat_id)

## 8. Seat Booking Flow

Step 1: User selects seats and system checks availability  
Step 2: Selected seats are temporarily locked  
Step 3: User completes payment  
Step 4: Booking is confirmed or seats are released

## 9. Preventing Double Booking

Problem:
- Multiple users attempt to book the same seat for the same show

Solution:

Redis-based Seat Lock
SETNX seat:showId:seatId bookingId EX 300

- Atomic seat lock
- Auto-release on timeout

Database Final Validation
INSERT INTO seat_booking(show_id, seat_id)
VALUES (?, ?)
ON CONFLICT DO NOTHING;

Redis ensures fast locking, database ensures final consistency.

## 10. API Design
GET /movies?city=  
GET /shows/{movieId}  
GET /seats/{showId}  
POST /bookings/lock  
POST /payments/pay  
POST /bookings/confirm  

## 11. Failure Handling
- Payment failure → release seat locks
- User timeout → auto-expire locks
- Redis failure → fallback to database locking
- Database failure → block bookings

## 12. Scalability Considerations
- Cache movies and shows in Redis
- Shard bookings by showId
- Horizontal backend scaling
- Async notifications (email/SMS)
- CDN for static content

## 13. Tra
