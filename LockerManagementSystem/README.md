# Locker Management System Design

## 1. Problem Statement
Design a locker management system for a warehouse where packages are
stored in lockers, customers receive a pickup code, and lockers are
allocated, accessed, and released safely without conflicts.

## 2. Assumptions
- Single warehouse location
- Fixed number of lockers
- Lockers have different sizes (Small, Medium, Large)
- One locker stores only one package at a time
- Each package is assigned to exactly one locker
- Customers receive a one-time pickup code
- Pickup code expires after package is collected
- No return handling in initial version
- System must handle concurrent package assignments
- Single timezone

## 3. Key Requirements

Functional Requirements
- Assign locker to incoming package
- Generate pickup code
- Allow customer to retrieve package using code
- Release locker after pickup
- View locker availability

Non-Functional Requirements
- No double allocation of lockers
- Concurrency safety
- Low latency
- High reliability

## 4. High-Level Architecture
Package Arrives → Locker Manager → Locker Assigned → Code Generated  
Customer → Enters Code → Locker Opens → Package Picked → Locker Released

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Concurrency: DB transactions / Java locks  
Authentication: Pickup code / OTP  
Deployment: AWS EC2  

(No frontend required initially)

## 6. Core Components

Locker
- lockerId
- size (S / M / L)
- status (AVAILABLE / OCCUPIED)

Package
- packageId
- size
- assignedLockerId
- pickupCode
- status (STORED / PICKED_UP)

Locker Manager
- Allocates lockers
- Releases lockers
- Validates pickup codes

## 7. Database Design (Short)

Locker
- locker_id (INT)
- size (ENUM)
- status (ENUM)

Package
- package_id (INT)
- locker_id (INT)
- pickup_code (VARCHAR)
- status (ENUM)
- created_at (TIMESTAMP)

Indexes:
- (size, status)
- (pickup_code)

## 8. Locker Allocation Logic

Rule:
- Assign the smallest available locker that fits the package size

Flow:
- Start database transaction
- Select available locker by size
- Lock row using FOR UPDATE
- Assign locker and generate pickup code
- Commit transaction

## 9. Preventing Double Locker Allocation

Problem:
- Multiple packages arrive concurrently and request lockers

Solution:
- Database transaction with row-level locking

SELECT * FROM locker
WHERE size >= :packageSize
AND status = 'AVAILABLE'
LIMIT 1
FOR UPDATE;

This ensures only one package gets the locker.

## 10. Pickup Flow
- Customer enters pickup code
- System validates the code
- If valid:
  - Open locker
  - Mark package as PICKED_UP
  - Mark locker as AVAILABLE

## 11. API Design
POST /lockers/assign  
GET /lockers/status  
POST /pickup/{code}  
GET /packages/{id}  

## 12. Failure Handling
- Invalid pickup code → reject request
- Database down → pause assignments
- Concurrent allocation → transaction rollback
- Code reuse → prevented via status check

## 13. Scalability Considerations
- Cache locker availability counts in Redis
- Scale backend horizontally
- Partition packages by date
- Async notifications via SMS or email

## 14. Trade-Offs
- Relational database ensures consistency
- Redis improves read performance
- Monolithic architecture is easy to maintain
- Simple allocation logic is predictable

## 15. Future Enhancements
- Multi-warehouse support
- IoT-based locker sensors
- Pickup expiry reminders
- Automatic return handling
- Mobile application
- QR-code based pickup
