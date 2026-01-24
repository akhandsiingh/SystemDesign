# Multi-Threaded Parking Lot System Design

## 1. Problem Statement
Design a parking lot system where multiple vehicles can enter and exit
at the same time, and the system must handle concurrency safely without
assigning the same parking spot to multiple vehicles.

## 2. Assumptions
- Multiple entry gates and exit gates
- Multiple vehicles can enter and exit concurrently
- One vehicle occupies one parking spot
- Parking spots are limited
- Pricing is time-based
- No advance booking
- Single parking location
- System runs 24×7
- Backend must be thread-safe

## 3. Why Multi-Threading Is Needed
- Multiple vehicles arrive at the same time
- Multiple entry gates assign parking spots concurrently
- Multiple exit gates release parking spots concurrently

Without synchronization:
- Two vehicles may get the same parking spot
- Incorrect fee calculation
- Data inconsistency

## 4. High-Level Architecture
Concurrent Flow:
Multiple Vehicles → Entry Gates (Threads) → Parking System  
Multiple Vehicles → Exit Gates (Threads) → Payment → Exit

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Concurrency: Java Threads, ExecutorService  
Database: MySQL  
Cache: Redis  
Locking: synchronized, ReentrantLock  
Deployment: AWS EC2  

## 6. Core Components (Thread-Aware)

Entry Gate (Runnable)
- Each gate runs in a separate thread
- Requests parking spot from Spot Manager

Exit Gate (Runnable)
- Runs in parallel
- Calculates fee and frees spot

ParkingSpotManager (Critical Section)
- Assigns and releases spots
- Must be thread-safe

## 7. Concurrency Design

Shared Resource:
- List / pool of available parking spots

Problem:
- Multiple threads trying to allocate or release spots at the same time

Solution:
- Use locking mechanisms
- Only one thread can modify spot availability at a time

## 8. Java Concurrency Approach

Option 1: synchronized
- Ensures only one thread enters the critical section
- Simple and easy to explain in interviews

Option 2: ReentrantLock
- More control over locking
- Explicit lock and unlock

## 9. Core Classes (Simplified)

ParkingSpot
- spotId
- type
- isAvailable

ParkingTicket
- ticketId
- entryTime
- exitTime
- spotId

ParkingSpotManager
- allocateSpot() → synchronized
- releaseSpot() → synchronized

## 10. Sample Multi-Threaded Logic

public synchronized ParkingSpot allocateSpot() {
    for (ParkingSpot spot : spots) {
        if (spot.isAvailable()) {
            spot.setAvailable(false);
            return spot;
        }
    }
    return null;
}

- Prevents two vehicles from getting the same spot
- Thread-safe allocation

## 11. Database Design

ParkingSpot
- spot_id (INT)
- spot_type (ENUM)
- is_available (BOOLEAN)

ParkingTicket
- ticket_id (INT)
- entry_time (TIMESTAMP)
- exit_time (TIMESTAMP)
- spot_id (INT)

## 12. Handling High Concurrency

Redis Atomic Operations
- INCR / DECR
- Prevents race conditions
- Faster than DB locking

Thread Pool
- Use ExecutorService
- Limits number of active threads
- Prevents system overload

## 13. Failure Scenarios & Handling

Two threads request same spot → Lock prevents conflict  
DB update failure → Rollback transaction  
System crash → DB maintains final state  
Redis failure → Fallback to database  

## 14. Trade-Offs
- synchronized → simple but less scalable
- ReentrantLock → more flexible
- Redis → fast but adds infrastructure
- Thread pools → controlled concurrency

## 15. Future Improvements
- Queue-based allocation (Kafka / SQS)
- Microservices for entry and exit
- Smart allocation using AI
- Multi-location parking support
