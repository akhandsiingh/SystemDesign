# Distributed Notification System – System Design

## 1. Problem Statement
Design a distributed notification system that can send notifications to users
at scale through multiple channels such as push notifications, email, SMS,
and in-app notifications. The system must be reliable, scalable, low latency,
and fault tolerant.

## 2. Assumptions
- Notifications can be real-time or scheduled
- Multiple backend services can trigger notifications
- Users can configure notification channel preferences
- External delivery providers may fail
- System must handle millions of notifications per day
- Notifications are event-driven
- Eventual consistency is acceptable

## 3. Key Requirements

Functional Requirements
- Send notifications to users
- Retry delivery on failure
- Support multiple delivery channels
- Store notification history
- Respect user channel preferences

Non-Functional Requirements
- High throughput
- Reliable delivery
- Horizontal scalability
- Fault tolerance

## 4. High-Level Architecture
Producer Services (Order / Ride / Payment)
                ↓
          Notification API
                ↓
           Message Queue (Kafka / RabbitMQ)
                ↓
      Notification Worker Services
                ↓
       External Providers (Push / Email / SMS)
                ↓
            User Devices

## 5. Relevant Tech Stack
Backend: Java Spring Boot  
Messaging: Kafka or RabbitMQ  
Database: PostgreSQL  
Cache: Redis (user preferences)  
Push Provider: Firebase FCM  
Email Provider: AWS SES  
SMS Provider: Twilio  
Scheduler: Cron or Quartz  
Deployment: AWS EC2  

## 6. Core Components

Notification Service
- Receives notification requests from producer services
- Validates payload and user data
- Publishes notification event to message queue

Message Queue
- Buffers traffic spikes
- Enables asynchronous processing
- Supports retry mechanisms
- Decouples business services from delivery logic

Worker Services
- Channel-specific processors (email, SMS, push)
- Consume events from queue
- Send notifications using external providers
- Update delivery status

Preference Service
- Stores user notification settings
- Filters delivery channels based on preferences

## 7. Database Design (Simplified)

Notification
- notification_id (BIGINT)
- user_id (BIGINT)
- channel (ENUM)
- message (TEXT)
- status (ENUM)
- created_at (TIMESTAMP)

UserPreference
- user_id (BIGINT)
- email_enabled (BOOLEAN)
- push_enabled (BOOLEAN)
- sms_enabled (BOOLEAN)

## 8. Real-Time Notification Flow

1. Business service triggers notification event
2. Notification API receives request
3. Event is published to message queue
4. Worker service consumes event
5. User preferences are checked
6. External provider is called
7. Delivery status is updated in database

This asynchronous flow improves scalability and reliability.

## 9. Retry Strategy

- If provider delivery fails, retry using exponential backoff
- Limit maximum retry attempts
- Failed messages are moved to a Dead Letter Queue
- Manual or automated recovery can process DLQ messages

## 10. Handling High Scale

- Partition queue by userId for parallel processing
- Deploy multiple worker instances
- Use idempotent notification identifiers
- Batch sending where supported (e.g., email campaigns)

## 11. Failure Handling

Scenario              Handling
Provider failure      Retry with backoff
Queue backlog         Scale worker services
Duplicate events      Idempotency checks
Database failure      Temporary buffering and retry

## 12. Trade-Offs
- Asynchronous queues improve reliability but introduce slight delays
- Eventual consistency is acceptable for notifications
- Separate channel workers increase flexibility
- Storing notification history improves auditing but increases storage usage

## Final Interview Closing Line
“I designed the notification system using an event-driven architecture
with a message queue to decouple producers from delivery workers. This
approach allows horizontal scaling, reliable retries, and multi-channel
notification delivery while maintaining low latency and fault tolerance.”