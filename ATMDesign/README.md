# ATM System – System Design

## 1. Problem Statement
Design an ATM system that allows customers to authenticate using a card
and PIN, perform banking transactions such as cash withdrawal and balance
inquiry, and safely dispense cash while ensuring security, consistency,
and fault tolerance.

## 2. Assumptions
- ATM is connected to a single bank in the initial version
- Only debit cards are supported
- ATM has limited cash
- PIN verification is done by the bank, not by the ATM
- Network connectivity is generally reliable but may fail
- No international transactions
- One transaction at a time per ATM
- Single currency

## 3. Key Requirements

Functional Requirements
- Card validation
- PIN verification
- Balance inquiry
- Cash withdrawal
- Cash deposit (optional)
- Receipt printing

Non-Functional Requirements
- Strong security
- Consistency (no double debit)
- High availability
- Fault tolerance
- Auditability

## 4. High-Level Architecture
ATM Machine → ATM Controller → Bank Core System → Account Database  
                                 ↓  
                           Transaction Ledger  

## 5. Relevant Technology
ATM Software: Embedded Java or C++  
Backend Services: Java Spring Boot  
Database: PostgreSQL  
Cache (optional): Redis  
Security: HSM, encryption  
Network: Secure TCP or HTTPS  

## 6. Core Components

ATM Machine
- Card reader
- Keypad
- Screen
- Cash dispenser
- Receipt printer

ATM Controller
- Orchestrates user flow
- Communicates with bank services
- Controls hardware operations safely

Bank Server
- Authentication service
- Account service
- Transaction service
- Ledger service

## 7. Database Design (Simplified)

Account
- account_id (BIGINT)
- balance (DECIMAL)
- status (ENUM)

Card
- card_number (VARCHAR)
- account_id (BIGINT)
- pin_hash (VARCHAR)

Transaction
- txn_id (BIGINT)
- account_id (BIGINT)
- type (ENUM)
- amount (DECIMAL)
- status (ENUM)
- created_at (TIMESTAMP)

Index:
- (account_id, created_at)

## 8. ATM Authentication Flow

1. User inserts card
2. ATM reads card number
3. User enters PIN
4. PIN is encrypted by ATM
5. Encrypted PIN sent to bank server
6. Bank validates PIN against stored hash
7. On success, transaction menu is shown
8. On failure, retries are limited and card may be blocked

ATM never stores or logs the PIN.

## 9. Cash Withdrawal Flow

1. User enters withdrawal amount
2. ATM checks it has sufficient cash
3. Bank checks account balance
4. Start database transaction
5. Debit account balance
6. Record transaction in ledger
7. Commit database transaction
8. ATM dispenses cash
9. Receipt is printed

## 10. Preventing Double Debit

Problem:
- Cash dispensed but network failure may cause duplicate debit

Solution:
- Use transaction state machine:
  INITIATED → DEBITED → CASH_DISPENSED → COMPLETED

Atomic debit operation:
UPDATE account
SET balance = balance - ?
WHERE account_id = ?
AND balance >= ?;

This guarantees exactly-once debit.

## 11. Cash Dispense Failure Handling

Scenario                        Handling
Debit success, cash failure     Auto-credit amount back
Cash success, debit failure     Block ATM and trigger audit
Network failure                 Retry safely using transaction ID
Power failure                   Recover using transaction status

The ledger is the source of truth.

## 12. Security Considerations
- Encrypted PIN blocks
- Secure communication channels
- Hardware Security Module (HSM)
- Daily withdrawal limits
- Card blocking after multiple failures
- Physical tamper detection

## 13. Concurrency Handling
- One active transaction per ATM
- Row-level locks on account records
- Serializable or repeatable-read isolation
- Idempotent transaction identifiers

## 14. Bank API Design

POST /auth/pin              → Verify PIN  
GET  /account/balance       → Balance inquiry  
POST /transaction/debit     → Withdraw money  
POST /transaction/credit    → Refund money  
GET  /transaction/status    → Recovery check  

## 15. Failure Handling and Recovery
- Every transaction has a unique ID
- Bank ledger is the final authority
- ATM synchronizes transaction state on restart
- Manual reconciliation reports are generated

## 16. Trade-Offs
- Centralized bank server ensures consistency
- Ledger-based design enables audits
- No caching for writes prioritizes correctness
- Slight latency is acceptable for financial safety
