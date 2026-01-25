# Meeting Room Reservation System Design – Recurring Meetings

## 1. Problem Statement
Extend the meeting room reservation system to support recurring meetings
(daily, weekly, monthly) while preventing conflicts and double booking,
even when multiple users book concurrently.

## 2. Assumptions
- Recurring meetings follow simple rules (daily / weekly / monthly)
- No complex rules such as last working day of the month
- All meetings use a single timezone
- Recurring meetings have start time, end time, recurrence pattern, and end date
- A recurring meeting can be cancelled entirely or per occurrence
- Entire series is rejected if any conflict exists
- Same concurrency requirements as single meetings

## 3. Key Requirements

Functional Requirements
- Create recurring meetings
- View recurring meetings
- Cancel entire series or single occurrence
- Prevent conflicts across all occurrences

Non-Functional Requirements
- Strong consistency
- Concurrency safety
- Acceptable performance

## 4. High-Level Architecture
User → Frontend → Backend → Recurrence Engine → Database  
(Optional) Email notification

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Scheduler: Java Time API  
Deployment: AWS EC2  

## 6. Core Design Decision

Store the recurrence rule and expand occurrences during booking.

Reasons:
- Easy to explain and implement
- Works well for office-scale systems
- Avoids storing excessive rows upfront

## 7. Data Model Design

MeetingRoom
- room_id (INT)
- name (VARCHAR)
- capacity (INT)

RecurringMeeting
- recurring_id (INT)
- room_id (INT)
- user_id (INT)
- start_time (TIME)
- end_time (TIME)
- recurrence_type (ENUM: DAILY, WEEKLY, MONTHLY)
- recurrence_interval (INT)
- start_date (DATE)
- end_date (DATE)
- status (ENUM: ACTIVE, CANCELLED)

BookingOccurrence
- occurrence_id (INT)
- recurring_id (INT)
- room_id (INT)
- start_datetime (TIMESTAMP)
- end_datetime (TIMESTAMP)
- status (ENUM: CONFIRMED, CANCELLED)

Index: (room_id, start_datetime, end_datetime)

## 8. Booking Flow for Recurring Meetings

Step 1: Generate all occurrences based on recurrence rule  
Step 2: Check conflicts for each occurrence using overlap condition  
(start1 < end2) AND (start2 < end1)  
Step 3: Use database transaction to insert recurring meeting and all occurrences

If any occurrence conflicts, reject the entire booking.

## 9. Handling Concurrent Requests

Problem:
- Multiple users create overlapping recurring meetings for the same room

Solution:
- Database transaction with row-level locking

SELECT * FROM booking_occurrence
WHERE room_id = ?
AND start_datetime < :end
AND end_datetime > :start
FOR UPDATE;

This prevents race conditions and ensures consistency.

## 10. Cancel Scenarios

Cancel Entire Series
- Update recurring meeting status to CANCELLED
- Update all related occurrences

Cancel Single Occurrence
- Mark the specific booking occurrence as CANCELLED

## 11. API Design
GET /rooms/available  
POST /recurring/bookings  
DELETE /recurring/{id}  
DELETE /occurrence/{id}  

## 12. Performance and Scalability
- Limit recurrence window (e.g., maximum one year)
- Cache availability data by date
- Batch insert occurrences
- Use indexes on time-based columns

## 13. Trade-Offs
- Expanding occurrences increases rows but simplifies logic
- Database locking ensures safety but adds slight latency
- Redis improves performance but is optional
- Monolithic design is easier for a fresher to maintain

## 14. Future Enhancements
- Support complex recurrence rules
- Timezone support
- Auto-release no-show meetings
- Calendar integrations
- Conflict resolution suggestions
