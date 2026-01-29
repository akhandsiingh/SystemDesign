# Fantasy Teams Leaderboard System Design

## 1. Problem Statement
Design a leaderboard system for fantasy sports where teams earn points
based on real match performance and users can see real-time rankings
during live matches.

Goals:
- Fast ranking updates
- Handle millions of users
- Show top performers instantly

## 2. Assumptions
- Each contest has many fantasy teams
- Points update frequently during live matches
- Leaderboard is read-heavy
- Ranking is based on total points
- Ties allowed but resolved by timestamp or team creation
- Points come from an external sports data feed
- Single region deployment initially

## 3. Key Requirements

Functional Requirements
- Create fantasy team
- Update team points
- Display leaderboard
- Show user rank
- Show top N teams

Non-Functional Requirements
- Very low latency
- High scalability
- Real-time updates
- Strong consistency for rankings

## 4. High-Level Architecture
Sports Data Feed → Points Service → Leaderboard Service → Redis → Users

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Cache: Redis Sorted Set  
Database: PostgreSQL  
Messaging: Kafka  
WebSocket: Real-time UI updates  
Deployment: AWS EC2  

## 6. Core Components

Contest
- contestId
- startTime
- endTime

FantasyTeam
- teamId
- userId
- contestId
- totalPoints

LeaderboardEntry
- teamId
- score
- rank

Points Processor
- Consumes sports feed
- Updates team scores

## 7. Database Design (Short)

FantasyTeam
- team_id (BIGINT)
- user_id (BIGINT)
- contest_id (BIGINT)
- total_points (DECIMAL)

Index:
- (contest_id)

## 8. Leaderboard Storage

Use Redis Sorted Set.

Key:
leaderboard:{contestId}

Example:
ZADD leaderboard:101 450 team123

- Score → points
- Member → teamId

Fetch top teams:
ZREVRANGE leaderboard:101 0 9 WITHSCORES

This enables fast ranking with logarithmic performance.

## 9. Real-Time Points Update Flow
- Sports API sends event
- Kafka receives the event
- Points Service calculates fantasy points
- Update Redis:

ZINCRBY leaderboard:101 6 team123

Ensures atomic updates and instant rank changes.

## 10. Finding User Rank
ZREVRANK leaderboard:101 team123

Returns rank with very low latency.

## 11. Handling High Concurrency
- Kafka buffers traffic spikes
- Redis supports fast writes
- Stateless backend services
- Batch updates when required

## 12. Tie-Breaking Strategy
If scores are equal:
- Use composite score with a small timestamp factor  
or  
- Maintain secondary ranking logic in the database.

## 13. API Design
GET /leaderboard/{contestId}  
GET /leaderboard/{contestId}/rank/{teamId}  
POST /teams  
POST /points/update  

## 14. Failure Handling
- Redis failure → rebuild leaderboard from database
- Kafka failure → retry queue
- Duplicate events → idempotent updates
- Database failure → continue cache operations

## 15. Scalability Considerations
- Redis Cluster for distributed ranking
- Partition leaderboards by contestId
- Horizontal backend scaling
- Read replicas for database
- Asynchronous persistence

## 16. Trade-Offs
- Redis is extremely fast but in-memory
- Database is durable but slower
- Kafka smooths traffic spikes
- Simple ranking logic is predictable

## 17. Future Enhancements
- Global leaderboard
- Friends-only leaderboard
- Automated prize distribution
- Anti-cheat detection
- Region-based ranking
