# OTP Service – Production-Grade System Design

## 1. Problem Statement
Design an OTP (One-Time Password) service that can securely generate,
deliver, and verify OTPs while preventing abuse, brute-force attacks,
and replay attacks, and scaling to millions of users.

## 2. Core Design Principles
A production-grade OTP system must be:
- Secure (no plaintext OTP storage)
- Single-use (OTP valid only once)
- Time-bound (short expiry)
- Rate-limited (prevent abuse)
- Fast (used in login and signup flows)
- Horizontally scalable (stateless service)

## 3. High-Level Architecture

Client  
↓  
Auth API  
↓  
OTP Service  
↓  
Redis (temporary OTP store)  
↓  
SMS / Email Provider  

Responsibilities:
- Auth API: validation and rate limiting
- OTP Service: OTP generation and verification
- Redis: temporary secure storage
- Provider: message delivery only

## 4. Key Assumptions
- OTP is numeric (6 digits)
- OTP expires in 2–5 minutes
- One active OTP per user per channel
- Redis is highly available
- SMS / Email providers may fail
- OTP requests can be abused

## 5. OTP Request Flow (Send OTP)

1. Client requests OTP  
   POST /auth/request-otp  

2. Auth API validates:
   - Phone/email format
   - User existence or signup eligibility
   - Rate limits  

3. OTP Service generates OTP using a secure random generator  

4. OTP is hashed using HMAC:
   hash = HMAC_SHA256(otp + secret)

5. Store in Redis:
   Key: otp:{userId}  
   Value:
   - hash
   - attemptsLeft
   - createdAt  
   TTL: 300 seconds  

6. OTP is sent asynchronously via SMS or Email provider  

7. API returns success (OTP is never returned)

## 6. Why Hash OTPs
OTP is a security secret.

If Redis is compromised:
- Plain OTP → account takeover
- Hashed OTP → unusable to attacker

Same principle as password storage.

## 7. OTP Verification Flow

1. Client submits OTP  
   POST /auth/verify-otp  

2. Fetch OTP data from Redis  

3. If key does not exist:
   - OTP expired or already used
   - Reject request  

4. Hash input OTP and compare with stored hash  

5. If match:
   - Authentication successful
   - Delete Redis key immediately
   - Issue session or token  

6. If mismatch:
   - Decrement attemptsLeft
   - If attemptsLeft reaches zero → delete OTP
   - Reject request  

## 8. One-Time Usage Guarantee
OTP is deleted immediately after successful verification to:
- Prevent replay attacks
- Ensure strict single-use behavior

## 9. Rate Limiting Strategy

OTP Request Rate Limit:
- Example: 3 requests per 10 minutes
- Redis key: otp:req:{userId}
- Use INCR with TTL

OTP Verification Attempt Limit:
- Example: maximum 5 attempts
- Stored inside OTP payload

Optional IP-based Rate Limiting:
- Implemented at API gateway
- Protects against bot attacks

## 10. Redis Data Model

Key: otp:{userId}  
TTL: 300 seconds  

Value:
- hash
- attemptsLeft
- createdAt  

Redis is used because:
- In-memory speed
- TTL support
- Atomic operations
- Fast deletion

## 11. Failure Handling

Scenario                     Handling
Redis down                   Fail OTP requests
SMS/Email provider fails     Retry or fallback
OTP expired                  Ask user to retry
Too many attempts            Lock OTP
Duplicate verification       Idempotent delete

## 12. Security Considerations
- Use cryptographically secure random generator
- OTP length ≥ 6 digits
- Never log OTP values
- Encrypt Redis at rest if possible
- Enforce HTTPS everywhere
- Use HMAC with secret salt
- Avoid sequential or predictable OTPs

## 13. Scalability
This design scales because:
- Redis operations are O(1)
- OTP service is stateless
- Horizontal scaling is easy
- Async delivery prevents blocking
- No database writes in hot path

Used in:
- Login
- Signup
- Payments
- Password reset

## 14. Trade-Offs
- Redis is volatile but acceptable for OTP
- Hashing adds CPU cost but ensures security
- TTL simplifies expiry handling
- Separating Auth and OTP services improves maintainability

## Final Interview Closing Line
“Although OTP looks simple, it is a security-critical system. I designed
it using Redis for temporary storage, cryptographic hashing for safety,
strict rate limiting to prevent abuse, and single-use deletion to avoid
replay attacks. This design is secure, scalable, and production-ready.”
