# Last Lap Hero – F1 Race Tracker System Design

## 1. Problem Statement
Design a real-time race tracking system that tracks driver positions,
updates lap times, displays a leaderboard, detects the fastest lap and
last lap hero, and pushes live updates to users with low latency.

## 2. Assumptions
- Data comes from an official race telemetry feed
- Approximately 20 drivers per race
- Millions of viewers make the system read-heavy
- Updates occur every 1–2 seconds
- Single race tracked initially
- No historical analytics in version one
- No betting features
- Global users require low latency

## 3. Key Requirements

Functional Requirements
- Show live driver positions
- Display lap times
- Identify fastest lap
- Highlight the last lap hero
- Provide a real-time leaderboard
- Notify major events such as overtakes or crashes

Non-Functional Requirements
- Ultra-low latency
- High availability
- Real-time streaming
- Scalable reads

## 4. High-Level Architecture
Telemetry Feed → Event Processor → Race Service → Redis  
→ WebSocket → Users

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Streaming: Kafka  
Cache: Redis  
Database: PostgreSQL  
Realtime Communication: WebSocket  
Deployment: AWS EC2 / Load Balancer  

Redis and WebSocket are critical for live updates.

## 6. Core Components

Driver
- driverId
- name
- team

Race
- raceId
- track
- totalLaps
- status

LapEvent
- driverId
- lapNumber
- lapTime
- position

LeaderboardEntry
- driverId
- currentPosition
- bestLap

## 7. Database Design (Short)

Driver
- driver_id (INT)
- name (VARCHAR)

Race
- race_id (INT)
- track (VARCHAR)

LapEvent
- event_id (BIGINT)
- race_id (INT)
- driver_id (INT)
- lap_time (FLOAT)
- position (INT)

Index:
- (race_id, driver_id)

## 8. Real-Time Update Flow
- Telemetry feed sends lap event
- Kafka receives the event
- Race processor:
  - Updates driver position
  - Updates fastest lap
  - Checks for last lap hero
- Update Redis leaderboard
- Push updates via WebSocket

Users see updates instantly.

## 9. Leaderboard Storage

Use Redis Sorted Set.

Key:
race:{raceId}:leaderboard

Example:
ZADD race:101 -1 driver44

- Score → negative position
- Member → driverId

Fetch leaders:
ZRANGE race:101 0 9

Provides millisecond-level ranking.

## 10. Detecting the Last Lap Hero
When the final lap starts:
- Track bestLastLapTime
- If a driver beats it, update hero

Store in Redis:
SET race:101:lastLapHero driver44

Allows constant-time lookup.

## 11. Handling Massive Read Traffic
- Redis caching for leaderboard data
- CDN for static assets
- WebSocket fanout for live updates
- Stateless backend services

## 12. Concurrency Handling
- Kafka partitions by raceId
- Atomic Redis updates
- Idempotent event processing
- Prevent duplicates using eventId

## 13. API Design
GET /race/live/{id}  
GET /race/{id}/hero  
GET /drivers  
POST /events  

## 14. Failure Handling
- Kafka failure → retry buffer
- Redis failure → fallback to database
- Duplicate telemetry → idempotent processing
- WebSocket disconnect → auto reconnect

## 15. Scalability Strategy
- Redis Cluster for distributed caching
- Kafka scaling for event ingestion
- Read replicas for database
- Geo-distributed CDN

Optimize for read-heavy workloads.

## 16. Trade-Offs
- Redis is extremely fast but in-memory
- Kafka adds infrastructure but handles spikes
- WebSocket enables real-time updates but increases connections
- Monolithic design initially simplifies debugging

## 17. Future Enhancements
- Multi-race dashboard
- Historical analytics
- AI-based race predictions
- Driver heatmaps
- Mobile push notifications
- Personalized fan views
