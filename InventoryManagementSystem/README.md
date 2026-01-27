## 1. Problem Statement
Design a system that handles orders and inventory where multiple users
can place orders concurrently while ensuring no overselling, correct
inventory updates, and thread-safe order processing.

## 2. Assumptions
- Single warehouse in initial version
- Fixed set of products
- Each product has limited stock
- Orders can be placed concurrently
- Inventory is updated only on order confirmation
- Payment is assumed successful
- System must be thread-safe
- Single currency
- No partial fulfillment

## 3. Key Requirements

Functional Requirements
- Place order
- Check inventory
- Reserve stock
- Confirm order
- Cancel order (optional)

Non-Functional Requirements
- Thread safety
- Strong consistency
- No overselling
- Scalability

## 4. High-Level Architecture
User → Order Service → Inventory Manager → Database  
(Orders processed concurrently using threads)

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Concurrency: Java Threads, ExecutorService  
Database: PostgreSQL  
Cache: Redis (optional)  
Messaging: Kafka (optional)  
Deployment: AWS EC2  

## 6. Core Components

Product
- productId
- name
- availableStock

Order
- orderId
- productId
- quantity
- status (CREATED / CONFIRMED / CANCELLED)

Inventory Manager
- Reserve stock
- Release stock
- Thread-safe updates

Order Processor
- Each order handled in a separate thread

## 7. Database Design (Short)

Product
- product_id (INT)
- stock (INT)

Order
- order_id (INT)
- product_id (INT)
- quantity (INT)
- status (ENUM)

Index:
- (product_id)

## 8. Why Multi-Threading Is Needed
- Multiple users place orders at the same time
- Multiple threads try to update inventory concurrently
- Without synchronization, overselling can occur

## 9. Concurrency Problem Example
- Stock = 5
- Thread A orders 3
- Thread B orders 3 at the same time

Without locking:
- Both succeed → stock becomes negative

With locking:
- Only one order succeeds

## 10. Thread-Safe Inventory Handling

Option 1: synchronized Method

public synchronized boolean reserveStock(Product product, int qty) {
    if (product.getStock() >= qty) {
        product.setStock(product.getStock() - qty);
        return true;
    }
    return false;
}

This approach is simple and prevents race conditions.

Option 2: ReentrantLock

lock.lock();
try {
    if (stock >= qty) {
        stock -= qty;
        return true;
    }
} finally {
    lock.unlock();
}

Provides more control over locking.

## 11. Multi-Threaded Order Processing Flow
- Order request arrives
- OrderProcessor thread starts
- Inventory is locked
- Stock is checked
- If sufficient, stock is reserved
- Order is created
- Lock is released

## 12. Using ExecutorService

ExecutorService executor = Executors.newFixedThreadPool(10);
executor.submit(new OrderProcessor(order));

This enables controlled concurrency and prevents thread explosion.

## 13. Database-Level Safety

UPDATE product
SET stock = stock - ?
WHERE product_id = ?
AND stock >= ?;

This ensures atomic inventory updates and prevents negative stock.

## 14. Handling Concurrent Orders (Summary)

Application Layer:
- synchronized methods or locks

Database Layer:
- Atomic conditional updates

Cache:
- Read-only optimization

## 15. Failure Handling
- Insufficient stock → reject order
- Thread crash → database rollback
- Duplicate order → idempotency key
- Partial update → transaction rollback

## 16. Scalability Considerations
- Partition inventory by productId
- Redis for read-heavy stock checks
- Kafka for async order processing
- Horizontal backend scaling

## 17. Trade-Offs
- synchronized is simple but slower
- Explicit locks provide more control
- Database atomic updates are safest
- Monolithic design simplifies debugging
