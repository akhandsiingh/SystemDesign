# File System Design (Unix / Linux Style)

## 1. Problem Statement
Design a file system that supports storing files and directories, allows
create, read, write, and delete operations, maintains a hierarchical
structure, enforces permissions, and efficiently manages disk space.

## 2. Assumptions
- Single machine (no distributed file system)
- Disk is divided into fixed-size blocks
- File system is persistent
- Directory structure is a tree
- Files are accessed using paths (e.g., /home/user/a.txt)
- No encryption or compression in the first version
- Single user (simplifies permissions)
- Basic crash recovery

## 3. Key Requirements

Functional Requirements
- Create file and directory
- Read file
- Write file
- Delete file or directory
- List directory contents

Non-Functional Requirements
- Efficient disk usage
- Fast lookup by path
- Data consistency
- Scalable to large disks

## 4. High-Level Architecture
User Command → File System API → Metadata Manager  
→ Disk Block Manager → Disk

## 5. Relevant Technology (Conceptual)
Language: C or Java  
Disk Access: Block-based I/O  
Data Structures: Trees, Maps  
OS Interface: POSIX-style APIs  

No database, cache, or backend services are used.

## 6. Core Components

Disk
- Divided into fixed-size blocks (e.g., 4KB)
- Stores actual file data

Inode
- Stores file metadata
- Does NOT store file name

Inode contains:
- inodeId
- file type (file or directory)
- file size
- permissions
- pointers to data blocks

Directory
- Special type of file
- Maps file name to inode number

Example:
home/
 ├── a.txt → inode 5
 └── b.txt → inode 9

Superblock
- Stores file system metadata
- Total disk size
- Number of blocks
- Free block information

## 7. Data Structures (Simplified)

Inode
- inodeId
- type (FILE / DIR)
- size
- permissions
- blockPointers[]

Directory Entry
- filename
- inodeId

Block Bitmap
- 1 → block used
- 0 → block free

Used to track free disk blocks.

## 8. File Creation Flow
1. Parse file path (e.g., /home/user/a.txt)
2. Traverse directories from root
3. Allocate a new inode
4. Allocate disk blocks
5. Add entry in parent directory
6. Update metadata

This operation is atomic.

## 9. File Read Flow
1. Resolve path to inode
2. Read inode metadata
3. Follow block pointers
4. Read data blocks
5. Return file content

## 10. File Write Flow
1. Resolve file inode
2. Check write permissions
3. Allocate additional blocks if required
4. Write data to blocks
5. Update inode size and metadata

## 11. Delete File Flow
1. Locate inode
2. Remove directory entry
3. Free data blocks
4. Free inode

Ensures no disk space leakage.

## 12. Path Resolution
Example path:
/home/user/docs/file.txt

Resolution:
root → home → user → docs → file.txt

At each step:
- Lookup name in directory
- Fetch inode
- Continue traversal

## 13. Permissions Model
- r → read
- w → write
- x → execute

Permissions are checked during every operation.

## 14. Space Management
Block Allocation Strategies:
- Contiguous allocation
- Linked allocation
- Indexed allocation (used here via inode pointers)

Indexed allocation is efficient and scalable.

## 15. Failure Handling
- Write data blocks before metadata
- Write metadata last
- Use basic write-ahead logging
- On crash, recover from log

(Journaling can be added in future versions.)

## 16. Time and Space Complexity
Operation           Complexity
Path Lookup         O(depth of path)
Read File           O(file size)
Write File          O(file size)
Create/Delete       O(1) metadata operations

## 17. Trade-Offs
- Inode-based design is scalable
- Block storage is space efficient
- Tree-based directories simplify lookup
- No indexing makes full search slower

## 18. Future Enhancements
- Journaling file system
- Page cache for faster reads
- Hard and soft links
- Compression
- Distributed file system support
- Snapshots and versioning

## Final Interview Closing Line
“I designed the file system using an inode-based architecture where
directories map file names to inodes, and inodes manage metadata and block
pointers. This design is simple, scalable, and closely matches how Unix
file systems work internally.”
