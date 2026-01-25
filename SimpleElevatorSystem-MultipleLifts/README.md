# Simple Elevator System Design (Multiple Lifts)

## 1. Problem Statement
Design a simple elevator system for a building with multiple elevators
that can handle multiple user requests efficiently and safely.

## 2. Assumptions
- Building has N floors (e.g., 10–20)
- Multiple elevators (e.g., 2–5)
- Each elevator can move up or down
- Each elevator serves all floors
- One request per button press
- No weight-based restrictions
- No emergency handling in initial version
- Simple scheduling (not AI-based)
- System runs continuously (24×7)

## 3. Key Requirements

Functional Requirements
- User can request elevator from any floor
- Elevator can move up or down
- Multiple elevators handle requests
- Elevator stops at requested floors

Non-Functional Requirements
- Correctness (no missed requests)
- Concurrency handling
- Fair scheduling
- Scalability (more elevators can be added)

## 4. High-Level Architecture
User presses button → Elevator Controller → Assign best elevator  
→ Elevator moves → Door opens

## 5. Relevant Tech Stack
Backend: Java  
Concurrency: Java Threads, ExecutorService  
Data Structures: Queue, PriorityQueue  
Synchronization: synchronized, ReentrantLock  
Deployment: JVM-based application  

(No database or frontend required)

## 6. Core Components

Elevator
- elevatorId
- currentFloor
- direction (UP / DOWN / IDLE)
- requestQueue

Elevator Controller
- Receives all requests
- Assigns requests to elevators
- Maintains elevator states

Request
- fromFloor
- toFloor
- direction

## 7. Elevator Scheduling Strategy
Nearest Elevator Algorithm:
- Choose elevator closest to request floor
- Prefer elevator moving in same direction
- If all busy, choose least loaded elevator

## 8. Multi-Threading Design

Why Multi-Threading
- Each elevator operates independently
- Multiple user requests arrive concurrently

Design
- Each elevator runs in its own thread
- Controller assigns requests
- Elevators process their own request queues

## 9. Thread-Safe Request Handling

Shared Resource
- Elevator request queues

Solution
- Synchronize request assignment
- Use thread-safe structures where required

## 10. Core Classes

Elevator
- elevatorId
- currentFloor
- direction
- requestQueue

ElevatorController
- listOfElevators
- assignRequest()

Request
- fromFloor
- toFloor
- direction

## 11. Sample Pseudo-Code

synchronized void assignRequest(Request req) {
    Elevator best = findNearestElevator(req);
    best.addRequest(req);
}

class Elevator implements Runnable {
    public void run() {
        while (true) {
            processNextRequest();
        }
    }
}

- Prevents race conditions
- Multiple elevators operate in parallel

## 12. Handling Concurrent Requests
- Central controller prevents duplicate assignment
- Requests handled via queues
- Fair scheduling avoids starvation
- No circular locks to prevent deadlocks

## 13. Edge Cases Considered
- Elevator idle at same floor
- Requests above and below current floor
- Elevator direction switching
- All elevators busy

## 14. Trade-Offs
- Simple scheduling for easier maintenance
- No database for lightweight design
- Threads provide realistic simulation
- No machine learning for predictability

## 15. Future Enhancements
- Weight sensors
- Emergency mode
- Express elevators
- Machine learning scheduling
- UI panel simulation
- Persistence using database
