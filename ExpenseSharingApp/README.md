# Expense Sharing System Design

## 1. Problem Statement
Design an expense sharing application where users can create groups, add
shared expenses, split amounts, and track who owes whom while keeping
balances accurate and consistent.

## 2. Assumptions
- Users are registered and authenticated
- Users can create groups (trip, flat, party, etc.)
- Expenses can be split equally, unequally, or by percentage
- All expenses are in a single currency
- Payments are settled offline (cash / UPI)
- No bank integration in initial version
- Single timezone
- Moderate number of users per group (≤ 50)

## 3. Key Requirements

Functional Requirements
- User registration and login
- Create and join groups
- Add shared expenses
- Split expenses among users
- View balances (who owes whom)
- Settle expenses (mark as paid)

Non-Functional Requirements
- Accuracy of balances
- Concurrency handling
- Low latency
- Easy scalability

## 4. High-Level Architecture
User → Web / Mobile App → Expense Service → Balance Calculator → Database  
(Optional) Notification Service

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Authentication: JWT  
Deployment: AWS EC2  

## 6. Core Components

User
- userId
- name
- email

Group
- groupId
- name
- members

Expense
- expenseId
- groupId
- paidBy (userId)
- totalAmount
- description

ExpenseSplit
- expenseId
- userId
- shareAmount

Balance
- userId
- owesUserId
- amount

## 7. Database Design (Short)

User
- user_id (INT)
- name (VARCHAR)

Group
- group_id (INT)
- name (VARCHAR)

Expense
- expense_id (INT)
- group_id (INT)
- paid_by (INT)
- total_amount (DECIMAL)

ExpenseSplit
- expense_id (INT)
- user_id (INT)
- amount (DECIMAL)

Balance
- user_id (INT)
- owes_user_id (INT)
- amount (DECIMAL)

Indexes:
- (group_id)
- (user_id, owes_user_id)

## 8. Expense Split Logic

Equal Split:
share = totalAmount / numberOfUsers

Unequal Split:
- User-defined share amounts

Percentage Split:
share = totalAmount × percentage / 100

## 9. Balance Update Logic

Example:
- A pays ₹600 for group of A, B, C
- Split equally → ₹200 each

Result:
- B owes A → ₹200
- C owes A → ₹200

Balance Update Flow:
- Start database transaction
- Insert expense and expense splits
- Update balance table
- Commit transaction

Ensures atomic and consistent updates.

## 10. Handling Concurrent Expense Updates

Problem:
- Multiple users add expenses concurrently in the same group

Solution:
- Use database transactions
- Lock balance rows using FOR UPDATE

SELECT * FROM balance
WHERE user_id IN (...)
FOR UPDATE;

## 11. Balance Simplification

Goal:
- Reduce number of transactions

Technique:
- Net balances between users

Example:
- A owes B ₹100
- B owes A ₹60
→ A owes B ₹40

## 12. API Design
POST /groups  
GET /groups/{id}  
POST /expenses  
GET /balances/{userId}  
POST /settle  

## 13. Failure Handling
- Invalid split → reject expense
- Database failure → rollback
- Duplicate request → idempotency key
- Partial update → transaction rollback

## 14. Scalability Considerations
- Cache balances in Redis
- Horizontal backend scaling
- Partition data by userId
- Async notifications

## 15. Trade-Offs
- PostgreSQL ensures correctness
- Redis improves read performance
- Monolithic design is simpler for a fresher
- No payment integration reduces complexity

## 16. Future Enhancements
- UPI / bank settlement
- Multi-currency support
- Expense attachments
- Monthly reports
- Smart debt simplification
- Offline-first mobile app
