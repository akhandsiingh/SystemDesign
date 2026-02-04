# File Collections Tracker – System Design

## 1. Problem Statement
Design a system that allows users to create collections of files, add or
remove files from collections, track file metadata and versions, view
collection history, and search files across collections.

The system stores only metadata and version information, not actual file
content.

## 2. Assumptions
- Single organization with multiple users
- Files are stored externally (S3 or local file system)
- System manages metadata only
- Files can belong to multiple collections
- Collections can be versioned
- No real-time collaboration in the first version
- Moderate scale (10^5–10^6 files)
- Single timezone

## 3. Key Requirements

Functional Requirements
- Create and delete file collections
- Add and remove files from collections
- Track file metadata (name, size, checksum)
- Track file versions
- View collection history
- Search files across collections

Non-Functional Requirements
- Consistency
- Fast reads
- Auditability
- Scalability

## 4. High-Level Architecture
User → Collection API → Metadata Service → Database  
(Optional) Storage Service → File System or S3

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Storage: AWS S3 or Local File System  
Search: PostgreSQL Full-Text Search  
Deployment: AWS EC2  

## 6. Core Components

File
- fileId
- name
- size
- checksum
- storagePath
- createdAt

FileVersion
- versionId
- fileId
- checksum
- createdAt

Collection
- collectionId
- name
- ownerId
- createdAt

CollectionFile
- collectionId
- fileId
- addedAt

## 7. Database Design (Simplified)

File
- file_id (BIGINT)
- name (VARCHAR)
- size (BIGINT)
- checksum (VARCHAR)
- path (TEXT)

FileVersion
- version_id (BIGINT)
- file_id (BIGINT)
- checksum (VARCHAR)
- created_at (TIMESTAMP)

Collection
- collection_id (BIGINT)
- name (VARCHAR)
- owner_id (BIGINT)

CollectionFile
- collection_id (BIGINT)
- file_id (BIGINT)

Indexes:
- (collection_id)
- (file_id)
- (name)

## 8. Core Flows

Create Collection
- User submits collection name
- Create collection record
- Return collectionId

Add File to Collection
- Validate file exists
- Insert mapping in CollectionFile
- Log audit event

A file can belong to multiple collections.

Update File (Versioning)
- Compute file checksum
- Compare with latest version
- If different, create new FileVersion
- Update metadata

All versions are preserved.

## 9. Collection Version Tracking
Purpose:
- Track which files existed in a collection at a given time

Approach:
- Store addedAt timestamps
- Track removal events
- Optionally snapshot collection state periodically

This enables audit and history reconstruction.

## 10. Search and Filtering
Supported Searches:
- File name
- Collection name
- File size range
- Date added

Implementation:
- PostgreSQL full-text search
- Indexed metadata fields

## 11. Handling Concurrency
Problem:
- Multiple users modifying the same collection simultaneously

Solution:
- Database transactions
- Optimistic locking using version columns
- Reject conflicting updates

## 12. API Design
POST /collections              → Create collection  
GET /collections/{id}          → View collection  
POST /collections/{id}/files   → Add file to collection  
GET /files/{id}/versions       → View file versions  
GET /search                    → Search files  

## 13. Failure Handling
- Missing file → mark as stale reference
- Duplicate add → ignore safely
- Database down → read-only mode
- Partial update → transaction rollback

## 14. Scalability Considerations
- Cache frequently accessed collections in Redis
- Paginate large collections
- Async audit logging
- Partition files by ownerId

## 15. Trade-Offs
- Metadata-only storage keeps system lightweight
- PostgreSQL provides strong consistency
- Redis improves read performance
- No real-time sync simplifies design

## 16. Future Enhancements
- Tags and labels
- File diff between versions
- Access control (ACLs)
- Multi-tenant isolation
- Backup and restore
- Event-based notifications
