# API Rate Limiter – System Design

## 1. Problem Statement
Design a rate limiter that restricts how many API requests a client
(user, IP, or API key) can make within a defined time window, rejects
excess requests, and operates with low latency at scale.

## 2. Assumptions
- Each request includes a client identifier (userId, API key, or IP)
- Limits are configurable per API
- System is distributed across multiple servers
- Redis is available for shared counters
- Rate limiting must be very fast (<1 ms)
- Hard limits only (no burst handling in v1)

## 3. Key Requirements

Functional Requirements
- Count requests per client
- Enforce limit per time window
- Reject excess requests with HTTP 429
- Support multiple APIs and limits

Non-Functional Requirements
- Low latency
- High accuracy
- Scalability
- Fault tolerance

## 4. High-Level Architecture
Client → API Gateway / Middleware → Rate Limiter → Backend Service  
                                   ↓  
                                 Redis  

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Cache: Redis  
Gateway: Nginx or API Gateway  
Deployment: AWS EC2  

## 6. Placement of Rate Limiter
Rate limiting is implemented at the API gateway or middleware layer
so that excessive traffic is blocked before reaching backend services.

## 7. Chosen Algorithm: Fixed Window Counter
Example limit:
100 requests per minute

Redis key format:
rate:{api}:{client}:{minute}

Example:
rate:/login:user123:2026-02-20T12:01

Redis operations:
INCR key
EXPIRE 60

If counter exceeds limit → reject request.

This approach is simple, fast, and widely used.

## 8. Request Flow

1. Request arrives at gateway
2. Extract client identifier and API name
3. Construct Redis key for current time window
4. Increment counter atomically
5. If count > limit → return HTTP 429
6. Otherwise → forward request to backend

## 9. Redis Data Model

Key:
rate:{api}:{client}:{timeWindow}

Value:
integer request count

TTL:
window duration (e.g., 60 seconds)

Redis provides:
- O(1) increments
- Automatic expiry
- Atomic operations

## 10. Distributed Server Handling
Multiple API servers share the same Redis store.
All servers increment the same key for a client,
ensuring consistent global rate limits.

## 11. Preventing Race Conditions
Redis INCR is atomic.

Typical pattern:
- INCR key
- If result == 1 → set EXPIRE

Or use Lua script to combine operations atomically.

## 12. Multiple Limits Example
A client may have:
- 100 requests per minute
- 1000 requests per hour

Store separate keys:
rate:min:{client}
rate:hour:{client}

Both limits must pass for request to proceed.

## 13. Failure Handling

Scenario              Handling
Redis down            Allow requests (fail-open)
Redis slow            Fallback local limit
Key missing           Treat as zero
Burst attack          Gateway block

## 14. Scalability Considerations
- Redis cluster for large scale
- Shard keys by clientId
- Stateless API servers
- Edge rate limiting via CDN

## 15. Trade-Offs
- Fixed window is simple but allows bursts at window edges
- Redis is fast but external dependency
- Centralized counting ensures accuracy
- Fail-open prioritizes availability

## 16. Other Algorithms (Brief)
Sliding Window:
More accurate but more complex.

Token Bucket:
Allows controlled bursts and smoothing.

For a fresher-level system, fixed window is sufficient.

## Final Interview Closing Line
“I implemented rate limiting at the API gateway using a Redis-backed
fixed-window counter. Each request increments a time-windowed key, and
excess requests are rejected with HTTP 429. This approach is simple,
low-latency, and scalable across distributed servers.”
