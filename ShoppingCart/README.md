# Shopping Cart System Design

## 1. Problem Statement
Design a shopping cart system that allows users to add products, update
quantities, remove items, and checkout while keeping the cart fast,
consistent, and reliable under concurrent updates.

## 2. Assumptions
- Users can be guest or logged-in
- One cart per user or session
- Products have limited stock
- Cart supports add, update, and remove operations
- Prices are fixed in the initial version
- Inventory is checked only during checkout
- Cart expires after inactivity (e.g., 30 minutes)
- Single currency
- No recommendations in initial version

## 3. Key Requirements

Functional Requirements
- Add item to cart
- Update item quantity
- Remove item from cart
- View cart
- Checkout cart

Non-Functional Requirements
- Low latency
- Consistency of quantity and price
- Concurrency safety
- Scalability

## 4. High-Level Architecture
User → Web / Mobile App → Cart Service → Redis Cache → Database  
Checkout → Order Service → Payment → Inventory

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Cache: Redis  
Database: PostgreSQL  
Authentication: JWT  
Payment: Razorpay  
Deployment: AWS EC2  

Cart data is primarily stored in Redis.

## 6. Core Components

Product
- productId
- name
- price
- stock

Cart
- cartId (userId or sessionId)
- list of cartItems

CartItem
- productId
- quantity
- priceAtAddTime

Order
- orderId
- userId
- totalAmount
- status

## 7. Data Storage Design

Redis (Cart Storage)

Key: cart:{userId}

{
  "productId1": { "qty": 2, "price": 500 },
  "productId2": { "qty": 1, "price": 1200 }
}

Features:
- Fast access
- Atomic operations
- Automatic expiry

PostgreSQL (Persistent Data)

Product
- product_id (INT)
- price (DECIMAL)
- stock (INT)

Order
- order_id (INT)
- user_id (INT)
- amount (DECIMAL)
- status (ENUM)

## 8. Cart Operations

Add to Cart
- Fetch cart from Redis
- Increase quantity
- Save back to Redis

Update Quantity
- Replace quantity value
- If quantity is zero, remove item

Remove Item
- Delete product entry from cart

All operations are constant time.

## 9. Handling Concurrent Cart Updates

Problem:
- Same user updates cart from multiple devices or tabs

Solution:
- Redis atomic operations
- Use HINCRBY for quantity updates

HINCRBY cart:user123 productId 1

Ensures thread-safe updates.

## 10. Checkout Flow
- Fetch cart from Redis
- Validate stock from database
- Calculate total price
- Create order inside DB transaction
- Reduce inventory
- Clear cart from Redis

Inventory is locked only during checkout.

## 11. Preventing Overselling

UPDATE product
SET stock = stock - ?
WHERE product_id = ?
AND stock >= ?;

This ensures atomic inventory reduction.

## 12. API Design
POST /cart/add  
PUT /cart/update  
DELETE /cart/remove  
GET /cart  
POST /checkout  

## 13. Failure Handling
- Redis failure → cart temporarily unavailable
- Payment failure → cart retained
- Stock shortage → checkout rejected
- Duplicate request → handled using idempotency key

## 14. Scalability Considerations
- Redis cluster for cart storage
- Horizontal backend scaling
- Cart sharding by userId
- CDN for product images

## 15. Trade-Offs
- Redis provides speed but is volatile
- Database used only at checkout ensures consistency
- Monolithic architecture is simpler for a fresher
- No inventory locking before checkout improves user experience

## 16. Future Enhancements
- Save-for-later feature
- Coupons and discounts
- Multiple carts per user
- Cross-device cart synchronization
- Abandoned cart reminders
