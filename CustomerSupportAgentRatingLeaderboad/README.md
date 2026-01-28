# Customer Support Agent Rating Leaderboard System Design

## 1. Problem Statement
Design a system that collects customer ratings for support agents and
displays a leaderboard ranking agents based on performance metrics such
as average rating, number of reviews, and recent activity.

## 2. Assumptions
- System is used within a single organization
- Fixed set of support agents
- Customers can rate an agent only after ticket resolution
- Rating scale is from 1 to 5
- One rating per ticket
- Leaderboard supports daily, weekly, and monthly views
- Internal access only (no public leaderboard)
- Single timezone
- Ratings are immutable once submitted

## 3. Key Requirements

Functional Requirements
- Submit rating for an agent
- Calculate average rating per agent
- Rank agents on leaderboard
- Filter leaderboard by time range
- View agent performance summary

Non-Functional Requirements
- Low latency leaderboard access
- Accurate ranking
- Concurrency handling
- Scalability for large number of ratings

## 4. High-Level Architecture
Customer → Rating Service → Rating Processor → Cache / Database  
→ Leaderboard UI

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Messaging: Kafka (optional)  
Authentication: JWT  
Deployment: AWS EC2  

## 6. Core Components

Agent
- agentId
- name
- team

Ticket
- ticketId
- agentId
- status

Rating
- ratingId
- ticketId
- agentId
- score (1–5)
- createdAt

LeaderboardEntry
- agentId
- avgRating
- totalRatings
- rank

## 7. Database Design (Short)

Agent
- agent_id (INT)
- name (VARCHAR)

Rating
- rating_id (INT)
- agent_id (INT)
- ticket_id (INT)
- score (INT)
- created_at (TIMESTAMP)

Constraints:
- Unique(ticket_id)

Indexes:
- (agent_id, created_at)

## 8. Rating Submission Flow
- Customer submits rating
- Validate ticket status is CLOSED
- Persist rating in database
- Trigger leaderboard update
- Return success response

## 9. Leaderboard Calculation Strategy

Ranking Metrics:
- Average rating
- Minimum rating count threshold (e.g., ≥ 10)
- Tie-breaker: total number of ratings

Leaderboard score = average rating

## 10. Leaderboard Implementation Using Redis

Redis Sorted Set:
ZADD agent_leaderboard avgRating agentId

Fetch top agents:
ZREVRANGE agent_leaderboard 0 9 WITHSCORES

This enables fast ranking and retrieval.

## 11. Handling Concurrent Ratings
- Database acts as the source of truth
- Redis leaderboard updated atomically
- Optional Kafka-based asynchronous processing

Ensures no race conditions during updates.

## 12. Time-Based Leaderboards
Maintain separate leaderboards:
- leaderboard:daily
- leaderboard:weekly
- leaderboard:monthly

Allows fast filtering by time range.

## 13. API Design
POST /ratings  
GET /leaderboard  
GET /agents/{id}/stats  
GET /leaderboard?range=weekly  

## 14. Failure Handling
- Duplicate rating → reject request
- Redis failure → compute leaderboard from database
- Database failure → block rating submission
- Partial update → retry or async reconciliation

## 15. Scalability Considerations
- Redis for fast leaderboard reads
- Database for long-term rating storage
- Kafka for async recalculation
- Horizontal scaling of backend services

## 16. Trade-Offs
- Redis is fast but volatile
- Database is accurate but slower
- Simple ranking logic is easy to explain
- No machine learning ensures predictable behavior

## 17. Future Enhancements
- Weighted ratings favoring recent reviews
- Combine CSAT with resolution time
- Team-level leaderboards
- Gamification badges
- AI-based sentiment analysis
