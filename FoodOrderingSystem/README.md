# Online Food Ordering System Design

## 1. Problem Statement
Design an online food ordering system where users can browse restaurants,
view menus, place orders, make payments, and track delivery while
handling high traffic during peak hours.

## 2. Assumptions
- System operates in multiple cities
- Each city has multiple restaurants
- Each restaurant has a fixed menu
- Users can place one order at a time
- Orders are paid online
- One delivery partner is assigned per order
- Delivery-only system (no dine-in)
- No surge pricing in initial version
- Single timezone per city

## 3. Key Requirements

Functional Requirements
- User registration and login
- Browse restaurants by city
- View menu and prices
- Add items to cart
- Place order and make payment
- Assign delivery partner
- Track order status

Non-Functional Requirements
- High availability
- Low latency
- Concurrency handling
- Scalability during peak hours

## 4. High-Level Architecture
User → Web / Mobile App → Order Service → Payment  
→ Restaurant → Delivery Partner → User

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Payment: Razorpay  
Messaging: Kafka  
Maps: Google Maps API  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- name
- address
- phone

Restaurant
- restaurantId
- name
- city
- isOpen

MenuItem
- itemId
- restaurantId
- price
- availability

Order
- orderId
- userId
- restaurantId
- totalAmount
- status (PLACED / PREPARING / OUT_FOR_DELIVERY / DELIVERED)

Delivery Partner
- partnerId
- currentLocation
- availability

## 7. Database Design (Short)

Restaurant
- restaurant_id (INT)
- city (VARCHAR)
- is_open (BOOLEAN)

MenuItem
- item_id (INT)
- restaurant_id (INT)
- price (DECIMAL)
- available (BOOLEAN)

Order
- order_id (INT)
- user_id (INT)
- restaurant_id (INT)
- status (ENUM)
- amount (DECIMAL)

Indexes:
- (restaurant_id)
- (user_id, status)

## 8. Order Placement Flow

Step 1: Add to Cart  
- Cart stored in Redis for fast access  

Step 2: Place Order  
- Validate menu availability  
- Calculate total amount  
- Create order with status PLACED  

Step 3: Payment  
- On success → confirm order  
- On failure → cancel order  

## 9. Handling Concurrency

Problem:
- Multiple users ordering limited-quantity items simultaneously

Solution:
- Check availability before confirmation
- Update inventory inside database transaction
- Optional Redis counters for popular items

## 10. Delivery Partner Assignment
- Find nearest available delivery partner
- Assign order
- Update status to OUT_FOR_DELIVERY
- Location calculated using Maps API

## 11. API Design
GET /restaurants?city=  
GET /menu/{restaurantId}  
POST /cart/add  
POST /orders  
GET /orders/{id}  
POST /delivery/assign  

## 12. Failure Handling
- Payment failure → cancel order
- Restaurant rejection → refund
- No delivery partner → retry or delay
- Service failure → graceful fallback

## 13. Scalability Considerations
- Cache menus aggressively in Redis
- Horizontal backend scaling
- Kafka for async order updates
- Partition orders by city
- CDN for static assets

## 14. Trade-Offs
- PostgreSQL ensures strong consistency
- Redis improves performance
- Monolithic architecture is simpler to maintain
- Kafka enables reliable asynchronous processing

## 15. Future Enhancements
- Surge pricing
- Scheduled orders
- Subscription plans
- AI-based recommendations
- Live delivery tracking
- Multi-restaurant carts
