# URL Shortener System Design

## 1. Problem Statement
Design a system that converts a long URL into a short URL and redirects
users from the short URL to the original long URL. The system should be
fast, reliable, and prevent collisions.

## 2. Assumptions
- System generates unique short URLs
- Short URLs are read-heavy (many redirects)
- No custom aliases in the initial version
- No authentication required
- URLs do not expire initially
- Single region deployment
- Redirects must be low latency
- Uses HTTP 301 or 302 redirects

## 3. Key Requirements

Functional Requirements
- Shorten a long URL
- Redirect short URL to long URL
- Track basic analytics (hit count)

Non-Functional Requirements
- Low latency redirects
- High availability
- Collision-free ID generation
- Read scalability

## 4. High-Level Architecture
Client → Shorten API → ID Generator → Database / Cache  
Client → Short URL → Redirect Service → Cache / Database → Long URL

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
ID Generation: Base62 Encoding  
Deployment: AWS EC2  

(No frontend required initially)

## 6. Core Components

URL Mapping
- shortCode
- longUrl
- createdAt
- hitCount

ID Generator
- Generates unique numeric ID
- Encodes ID using Base62

Redirect Service
- Reads shortCode
- Redirects to longUrl

## 7. Database Design (Short)

UrlMapping
- id (BIGSERIAL)
- short_code (VARCHAR)
- long_url (TEXT)
- hit_count (BIGINT)
- created_at (TIMESTAMP)

Indexes:
- Unique(short_code)

## 8. Short URL Generation Logic

Step 1: Generate unique ID using database auto-increment  
Step 2: Encode ID using Base62  

Example:
- ID = 125
- shortCode = "cb"

This approach is deterministic and collision-free.

## 9. Redirect Flow
- User accesses short URL
- Check Redis cache for shortCode
- If found, redirect immediately
- If not found:
  - Fetch mapping from database
  - Cache result in Redis
  - Redirect user

Redis handles most of the redirect traffic.

## 10. Preventing Collisions
- Unique database constraint on short_code
- Base62 encoding of unique numeric ID
- No random code generation

## 11. API Design
POST /shorten  
GET /{shortCode}  
GET /stats/{shortCode}  

## 12. Handling High Traffic
- Cache popular URLs in Redis
- Stateless backend services
- Horizontal scaling
- Optional CDN for redirect domain

## 13. Failure Handling
- Redis failure → read from database
- Database failure → disable shorten API
- Invalid short code → return 404
- Duplicate request → optional idempotency handling

## 14. Trade-Offs
- Database ID + Base62 encoding is simple and safe
- Redis improves redirect performance
- No custom aliases avoids conflicts
- Monolithic design is easy to maintain

## 15. Future Enhancements
- Custom aliases
- URL expiry
- User accounts
- Advanced analytics
- QR code support
- Geo-based redirects
