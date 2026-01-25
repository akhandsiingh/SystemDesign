# Train Platform Management System Design

## 1. Problem Statement
Design a system to manage train arrivals and departures at a railway
station, assign platforms, avoid conflicts, and handle delays efficiently.

## 2. Assumptions
- Single railway station
- Fixed number of platforms
- Trains have scheduled arrival and departure times
- One train occupies one platform at a time
- A platform cannot be shared simultaneously
- Train delays are possible
- No ticketing or passenger reservation
- System handles multiple trains concurrently
- Single timezone

## 3. Key Requirements

Functional Requirements
- Assign platform to arriving train
- Prevent multiple trains from using the same platform
- Handle train delays
- Release platform after departure
- View current platform status

Non-Functional Requirements
- Strong consistency
- Concurrency safety
- Low response time
- Scalability for more trains or platforms

## 4. High-Level Architecture
Train Schedule → Platform Manager → Platform Assignment  
→ Arrival → Departure → Release Platform

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Concurrency: DB transactions / Java locks  
Scheduler: Spring Scheduler  
Deployment: AWS EC2  

## 6. Core Components

Train
- trainId
- trainNumber
- arrivalTime
- departureTime
- status (SCHEDULED / ARRIVED / DEPARTED / DELAYED)

Platform
- platformId
- isAvailable
- currentTrainId

Platform Manager
- Assigns platforms
- Releases platforms
- Handles conflicts and delays

## 7. Database Design (Short)

Platform
- platform_id (INT)
- is_available (BOOLEAN)
- current_train_id (INT)

TrainSchedule
- train_id (INT)
- arrival_time (TIMESTAMP)
- departure_time (TIMESTAMP)
- platform_id (INT)
- status (ENUM)

Index: (arrival_time, departure_time, platform_id)

## 8. Platform Allocation Logic

Rule:
Assign a platform where:
existing.departure_time <= new.arrival_time

Algorithm:
- Check available platforms
- Lock selected platform
- Assign train
- Update database

## 9. Handling Concurrent Arrivals

Problem:
- Multiple trains arrive at the same time requesting platforms

Solution:
- Use database transaction with row-level locking

SELECT * FROM platform
WHERE is_available = true
LIMIT 1
FOR UPDATE;

This ensures only one train gets a platform at a time.

## 10. Delay Handling
- Update departure time
- Recalculate conflicts
- Reassign platforms if required
- Maintain data consistency

## 11. API Design
POST /trains/schedule  
GET /platforms/status  
POST /trains/arrive/{id}  
POST /trains/depart/{id}  
PUT /trains/delay/{id}  

## 12. Failure Handling
- Database failure → pause scheduling
- Platform conflict → retry assignment
- Train delay → dynamic rescheduling

## 13. Scalability Considerations
- Cache platform availability in Redis
- Scale backend horizontally
- Partition schedules by date
- Async notifications for updates

## 14. Trade-Offs
- Relational database ensures strong consistency
- Simple allocation logic is predictable
- Redis improves read performance
- Monolithic design is easy to maintain

## 15. Future Enhancements
- Multi-station support
- Priority handling for express trains
- Passenger display boards
- AI-based scheduling
- Real-time GPS integration
