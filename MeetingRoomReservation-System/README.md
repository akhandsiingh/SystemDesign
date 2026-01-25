# Meeting Room Reservation System Design

## 1. Problem Statement
Design a system that allows employees to view, book, update, and cancel
meeting room reservations while preventing double booking of the same
room for the same time slot.

## 2. Assumptions
- Used within a single organization
- Fixed number of meeting rooms
- Each room has a capacity
- Meetings are booked for a time interval (start–end)
- Time slots are in minutes
- No recurring meetings in the initial version
- Only authenticated users can book rooms
- System must handle concurrent booking requests
- Single timezone (office timezone)

## 3. Key Requirements

Functional Requirements
- View available meeting rooms for a time range
- Book a meeting room
- Cancel or update a booking
- Prevent overlapping bookings

Non-Functional Requirements
- Consistency (no double booking)
- Concurrency handling
- Low latency
- Simple scalability

## 4. High-Level Architecture
User → Web App → Backend Service → Database  
(Optional) Notification → Email Service

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Notifications: SMTP / Email API  
Deployment: AWS EC2  

## 6. Core Components

Meeting Room
- roomId
- name
- capacity
- location

Booking
- bookingId
- roomId
- userId
- startTime
- endTime
- status (CONFIRMED / CANCELLED)

Availability Service
- Checks overlapping bookings
- Returns available rooms

## 7. Database Design

MeetingRoom
- room_id (INT)
- name (VARCHAR)
- capacity (INT)

Booking
- booking_id (INT)
- room_id (INT)
- user_id (INT)
- start_time (TIMESTAMP)
- end_time (TIMESTAMP)
- status (ENUM)

Index: (room_id, start_time, end_time)

## 8. Preventing Double Booking

Overlap Condition:
(start1 < end2) AND (start2 < end1)

Booking Flow:
- Start database transaction
- Check for overlapping bookings
- Reject if conflict exists
- Insert booking and commit transaction

## 9. Handling Concurrent Requests

Problem:
- Two users attempt to book the same room at the same time

Solutions:
- Database transaction with row-level locking
- Optimistic locking using version column
- Redis-based distributed locking

For fresher-level design, database locking is sufficient.

## 10. API Design
GET /rooms/available  
POST /bookings  
DELETE /bookings/{id}  
GET /bookings/user/{id}  

## 11. Scalability Considerations
- Cache availability data in Redis
- Scale backend horizontally
- Partition bookings by date if required

## 12. Failure Handling
- Database failure → reject booking
- Concurrent conflict → retry or fail gracefully
- Email failure → booking still succeeds

## 13. Trade-Offs
- PostgreSQL ensures strong consistency
- Monolithic architecture for simplicity
- Redis added only for performance
- Avoids microservices complexity

## 14. Future Enhancements
- Recurring meetings
- Calendar integrations
- Room equipment filtering
- Auto-release unused bookings
- Analytics dashboard
