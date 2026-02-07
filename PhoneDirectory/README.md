# Phone Directory System – System Design

## 1. Problem Statement
Design a phone directory system that allows users to store contacts and
perform fast searches by name or phone number, along with basic CRUD
operations such as add, update, delete, and list contacts.

## 2. Assumptions
- Single user per directory (personal phonebook)
- A contact can have multiple phone numbers
- Phone numbers are unique across contacts
- System is read-heavy (frequent searches)
- No cloud sync in the first version
- No spam detection
- Name search is case-insensitive

## 3. Key Requirements

Functional Requirements
- Add a contact
- Search contact by name
- Search contact by phone number
- Update contact details
- Delete contact
- List all contacts

Non-Functional Requirements
- Fast lookup
- Low memory usage
- Data consistency
- Easy scalability

## 4. High-Level Architecture
User → Directory API → In-Memory Index / Cache → Database

## 5. Relevant Tech Stack
Frontend: React  
Backend: Java Spring Boot  
Database: PostgreSQL  
Cache: Redis  
Search Index: Trie or DB indexes  
Deployment: AWS EC2  

## 6. Core Components

Contact
- contactId
- name
- phoneNumbers
- email (optional)

PhoneDirectory
- Stores all contacts
- Manages indexing and lookup

Search Index
- Name-based index
- Phone-number-based index

## 7. Database Design (Simplified)

Contact
- contact_id (BIGINT)
- name (VARCHAR)
- email (VARCHAR)

PhoneNumber
- phone_id (BIGINT)
- contact_id (BIGINT)
- phone_number (VARCHAR, unique)

Indexes:
- phone_number (UNIQUE)
- name

## 8. Search Design

Search by Phone Number
- Use database index or HashMap-style lookup
- O(1) search
- Phone number uniquely identifies a contact

Search by Name
Option 1: Database query
SELECT * FROM contact WHERE name ILIKE 'rah%';

Option 2: Trie-based prefix search
- Faster auto-suggestions
- Better user experience

## 9. Add Contact Flow
1. Validate phone number uniqueness
2. Save contact in database
3. Insert phone numbers
4. Update name and phone indexes
5. Cache frequently accessed contacts

## 10. Update and Delete Contact Flow
- Update or delete record in database
- Update search indexes
- Invalidate related cache entries

This ensures consistency across DB, cache, and indexes.

## 11. Handling Duplicate Names
- Duplicate names are allowed
- Phone number acts as the unique identifier
- Search by name may return multiple contacts

## 12. API Design
POST   /contacts                  → Add contact  
PUT    /contacts/{id}              → Update contact  
DELETE /contacts/{id}              → Delete contact  
GET    /contacts/search?name=      → Search by name  
GET    /contacts/search?phone=     → Search by phone  

## 13. Failure Handling
- Duplicate phone number → reject request
- Database down → read-only mode
- Cache down → fallback to database
- Invalid input → validation error

## 14. Scalability Considerations
- Cache frequently accessed contacts in Redis
- Paginate contact listings
- Index search-heavy fields
- Stateless backend for horizontal scaling

## 15. Trade-Offs
- PostgreSQL provides reliable persistent storage
- Redis improves read performance
- Trie improves search UX but consumes memory
- Monolith design is simpler to build and maintain

## 16. Future Enhancements
- Cloud sync across devices
- Spam detection
- Contact groups
- Favorites
- Import and export contacts
- Tag-based search
