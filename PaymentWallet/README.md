# Digital Payment Wallet System Design

## 1. Problem Statement
Design a digital payment wallet that allows users to store money, send
and receive payments, pay merchants, and view transaction history while
ensuring security, consistency, and reliability.

## 2. Assumptions
- Users must register and verify email or phone
- Single currency in the initial version
- Wallet holds preloaded balance
- Supports peer-to-peer and merchant payments
- Bank integration only for add and withdraw money
- No credit or loan features initially
- Single timezone
- System handles high concurrency
- Strong consistency is required for money movement

## 3. Key Requirements

Functional Requirements
- User registration and login
- Add money to wallet
- Send money to another user
- Pay merchants
- Withdraw money to bank
- View transaction history

Non-Functional Requirements
- Exactly-once money transfer
- High security
- Low latency
- Fault tolerance
- Auditability

## 4. High-Level Architecture
User → Wallet App → Wallet Service → Ledger → Bank / Payment Gateway  
Notifications handled asynchronously

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Messaging: Kafka  
Authentication: JWT + 2FA  
Payment Gateway: Razorpay / Bank APIs  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- email
- phone
- status (ACTIVE / BLOCKED)

Wallet
- walletId
- userId
- balance
- currency

Transaction
- transactionId
- fromWalletId
- toWalletId
- amount
- type (ADD / TRANSFER / PAY / WITHDRAW)
- status (SUCCESS / FAILED)

Ledger
- Immutable record of all wallet balance changes

## 7. Database Design (Short)

Wallet
- wallet_id (BIGINT)
- user_id (BIGINT)
- balance (DECIMAL)
- currency (VARCHAR)

Transaction
- transaction_id (BIGINT)
- from_wallet (BIGINT)
- to_wallet (BIGINT)
- amount (DECIMAL)
- status (ENUM)
- created_at (TIMESTAMP)

Indexes:
- (from_wallet, created_at)
- (to_wallet, created_at)

Ledger
- ledger_id (BIGINT)
- wallet_id (BIGINT)
- transaction_id (BIGINT)
- debit (DECIMAL)
- credit (DECIMAL)
- balance_after (DECIMAL)

Ledger is immutable and fully auditable.

## 8. Money Transfer Flow

Example: User A sends ₹500 to User B

- Start database transaction
- Lock both wallets using FOR UPDATE
- Check sufficient balance in sender wallet
- Debit sender wallet
- Credit receiver wallet
- Insert ledger entries
- Insert transaction record
- Commit transaction

Ensures atomic and consistent money movement.

## 9. Preventing Double Spending

Problem:
- Same transfer request retried due to network failure

Solution:
- Idempotency key per request
- Transaction status validation
- Conditional wallet update

UPDATE wallet
SET balance = balance - ?
WHERE wallet_id = ?
AND balance >= ?;

Ensures exactly-once execution and prevents negative balance.

## 10. Handling Concurrent Transfers
- Row-level locking on wallet records
- Serializable or repeatable-read isolation
- Redis used only for read optimization
- Database remains the source of truth

## 11. Add Money and Withdraw Flow

Add Money
- Call payment gateway
- On success, credit wallet
- Create ledger and transaction records

Withdraw Money
- Debit wallet
- Call bank API
- Update transaction status

## 12. API Design
GET /wallet/balance  
POST /wallet/add  
POST /wallet/transfer  
POST /wallet/withdraw  
GET /transactions  

## 13. Failure Handling
- Payment gateway failure → retry or rollback
- Database failure → abort transaction
- Duplicate request → idempotency handling
- Partial update → automatic rollback

## 14. Scalability Considerations
- Kafka for asynchronous notifications
- Read replicas for transaction history
- Redis for hot balance reads
- Horizontal scaling of backend services

## 15. Trade-Offs
- PostgreSQL prioritizes correctness over speed
- Ledger adds storage cost but ensures audit safety
- Redis improves performance but is not authoritative
- Monolithic design simplifies money flow reasoning

## 16. Future Enhancements
- Multi-currency support
- International transfers
- QR-based payments
- Credit or BNPL features
- Fraud detection
- Merchant dashboards
