# Online Stock Trading Platform System Design

## 1. Problem Statement
Design an online stock trading platform that allows users to view market
data, place buy and sell orders, execute trades, and track portfolio and
funds while ensuring correctness, low latency, and no double execution.

## 2. Assumptions
- Platform supports equity trading only
- Market hours are fixed
- Supported order types: Market and Limit
- Single exchange integration (e.g., NSE)
- Users must be KYC verified
- Funds are preloaded before trading
- Single timezone
- Real-time prices via market data feed
- High concurrency during market hours

## 3. Key Requirements

Functional Requirements
- User login and KYC verification
- View live stock prices
- Place buy and sell orders
- Order matching and execution
- View order status
- Portfolio and PnL tracking
- Funds and holdings view

Non-Functional Requirements
- Low latency
- High consistency
- High availability
- Concurrency safety
- Fault tolerance

## 4. High-Level Architecture
User → Trading App → Order Management System (OMS)  
OMS → Exchange → Trade Execution  
Market Feed → Price Service → User Interface

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Market Feed: WebSocket  
Database: PostgreSQL  
Cache: Redis  
Messaging: Kafka  
Authentication: JWT  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- kycStatus
- availableFunds

Order
- orderId
- userId
- stockSymbol
- quantity
- price
- type (BUY / SELL)
- status (PLACED / EXECUTED / CANCELLED)

Trade
- tradeId
- buyOrderId
- sellOrderId
- executedPrice
- quantity

Portfolio
- userId
- stockSymbol
- quantity
- avgPrice

## 7. Database Design (Short)

Order
- order_id (BIGINT)
- user_id (BIGINT)
- symbol (VARCHAR)
- type (ENUM)
- price (DECIMAL)
- quantity (INT)
- status (ENUM)

Index:
- (symbol, status)

Trade
- trade_id (BIGINT)
- buy_order_id (BIGINT)
- sell_order_id (BIGINT)
- price (DECIMAL)
- quantity (INT)

Portfolio
- user_id (BIGINT)
- symbol (VARCHAR)
- quantity (INT)
- avg_price (DECIMAL)

## 8. Order Placement Flow

Step 1: Order Validation
- Check KYC status
- Check available funds for BUY
- Check holdings for SELL

Step 2: Place Order
- Save order with status PLACED
- Publish order event to Kafka

Step 3: Order Matching
- Exchange matches the order
- Execution result is received

Step 4: Trade Confirmation
- Update order status
- Update portfolio
- Update available funds

## 9. Preventing Double Execution

Problem:
- Same order processed multiple times due to retries or failures

Solution:
- Idempotent order IDs
- Order status state machine
- Database conditional update

UPDATE orders
SET status = 'EXECUTED'
WHERE order_id = ?
AND status = 'PLACED';

Ensures exactly-once execution.

## 10. Handling High Concurrency
- In-memory order queues
- Kafka for asynchronous processing
- Redis caching for frequently accessed data
- Horizontal scaling of OMS

## 11. Real-Time Market Data Handling
- Market feed via WebSocket
- Update prices in Redis cache
- Push updates to users via WebSocket

Ensures low latency and high throughput.

## 12. API Design
GET /stocks/prices  
POST /orders  
GET /orders/{id}  
GET /portfolio  
GET /funds  

## 13. Failure Handling
- Exchange timeout → retry
- Duplicate order → idempotency
- Kafka failure → retry queue
- Database failure → stop trading

## 14. Scalability Considerations
- Kafka partitions by stock symbol
- Redis for market data caching
- Stateless backend services
- Separate read and write databases

## 15. Trade-Offs
- Monolithic OMS for simplicity
- Kafka for reliable async processing
- Redis for fast price updates
- Avoid microservices to reduce complexity

## 16. Future Enhancements
- Options and futures trading
- Algorithmic trading APIs
- AI-based alerts
- Multi-exchange support
- Margin trading
- Advanced charting
