# Dictionary Application – System Design

## 1. Problem Statement
Design a dictionary application that stores words and their meanings,
supports fast word lookup, and optionally handles synonyms, examples,
and admin updates.

## 2. Assumptions
- Single language in the first version (e.g., English)
- Words are unique and case-insensitive
- Workload is read-heavy (many searches)
- Meanings do not change frequently
- No pronunciation audio in the first version
- Only admins can add or update words

## 3. Key Requirements

Functional Requirements
- Search a word and get its meaning
- List synonyms or examples (optional)
- Add or update words (admin only)
- Prefix search or auto-suggest (optional)

Non-Functional Requirements
- Low latency lookups
- High availability
- Scalable reads
- Strong consistency for updates

## 4. High-Level Architecture
Client → Dictionary API → Cache (Redis) → Database  
(Optional) Trie or Search Engine for auto-suggestions

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Search (optional): Trie or PostgreSQL LIKE / FTS  
Deployment: AWS EC2  

## 6. Core Components

Word
- wordId
- text
- pronunciation (optional)
- createdAt

Meaning
- meaningId
- wordId
- definition
- partOfSpeech
- example (optional)

Synonym (optional)
- wordId
- synonymWordId

## 7. Database Design (Simplified)

Word
- word_id (BIGINT)
- text (VARCHAR, unique, lowercase)

Meaning
- meaning_id (BIGINT)
- word_id (BIGINT)
- definition (TEXT)
- part_of_speech (VARCHAR)
- example (TEXT)

Indexes:
- word(text) UNIQUE
- meaning(word_id)

## 8. Read Path (Most Important)

The dictionary is read-heavy, so the read path is optimized.

Flow:
1. Client searches for a word
2. API checks Redis cache
3. If cache hit → return meaning
4. If cache miss:
   - Query database
   - Store result in Redis
   - Return response

This ensures O(1) lookup for most requests.

## 9. Write Path (Admin Updates)

Flow:
1. Admin adds or updates a word
2. Write data to database using a transaction
3. Invalidate Redis cache for that word
4. Future reads fetch fresh data

This keeps data consistent and simple to manage.

## 10. Prefix Search / Auto-Suggest (Optional)

Option 1: Trie (In-Memory)
- Fast prefix search
- Built at startup
- Suitable for moderate vocabulary size

Option 2: Database Query
SELECT text FROM word WHERE text LIKE 'pre%';

Both approaches are acceptable for a fresher-level design.

## 11. API Design
GET  /words/{text}        → Get word meaning  
GET  /search?prefix=     → Auto-suggest words  
POST /admin/words        → Add new word  
PUT  /admin/words/{id}   → Update word  

## 12. Failure Handling
- Redis down → fallback to database
- Database down → read-only mode
- Word not found → return 404
- Duplicate word → reject request

## 13. Scalability Considerations
- Cache popular words aggressively
- Stateless backend for horizontal scaling
- Use read replicas for database
- Pre-warm cache for common words

## 14. Trade-Offs
- PostgreSQL provides structured and reliable storage
- Redis enables ultra-fast reads
- Trie improves prefix search but uses more memory
- Monolith architecture is simpler to maintain

## 15. Future Enhancements
- Multi-language support
- Audio pronunciation
- User bookmarks
- Offline mode
- Daily word notifications
- ML-based meaning suggestions
