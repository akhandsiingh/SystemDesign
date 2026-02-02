# Unix Find Command – File Search System Design

## 1. Problem Statement
Design a system similar to the Unix `find` command that searches files
and directories starting from a given root directory and returns file
paths that match specific conditions such as name, type, size, modified
time, or permissions.

## 2. Assumptions
- Runs on a local file system
- Directory structure forms a tree
- Search starts from a given root directory
- File system size can be very large
- Single machine (no distributed filesystem)
- Read-only search (no file modification)
- Symbolic links are ignored in the initial version
- Permissions are already enforced by the OS

## 3. Key Requirements

Functional Requirements
- Recursively traverse directories
- Filter files based on multiple conditions
- Support combining filters
- Print full file paths of matching files

Non-Functional Requirements
- Efficient traversal
- Low memory usage
- Handle deep directory structures
- Avoid stack overflow

## 4. High-Level Architecture
Start Directory → Traversal Engine → Filter Engine → Output Formatter

## 5. Relevant Tech Stack
Language: C or Java  
File API: POSIX APIs or Java NIO  
Traversal Algorithm: DFS / BFS  
Filter Design: Predicate Pattern  

No database, cache, or backend services required.

## 6. Core Components

FileNode
- path
- name
- type (file / directory)
- size
- permissions
- lastModifiedTime

Traversal Engine
- Walks the directory tree
- Emits file nodes

Filter Engine
- Applies filter conditions
- Determines whether a file matches

Output Formatter
- Prints matching file paths

## 7. Directory Traversal Logic

Chosen Approach: Iterative DFS

Reason:
- Avoids stack overflow
- Uses explicit stack
- Memory efficient for deep directory trees

Pseudo Code:

Stack<File> stack = new Stack<>();
stack.push(rootDir);

while (!stack.isEmpty()) {
    File current = stack.pop();

    if (current.isDirectory()) {
        for (File f : current.listFiles()) {
            stack.push(f);
        }
    } else {
        processFile(current);
    }
}

## 8. Filter Design

Use Predicate Pattern

Each filter implements:
boolean matches(FileNode file);

Example Filters:
- Name filter (-name "*.txt")
- Size filter (-size > 10MB)
- Type filter (-type f)

Filters are combined using AND logic by default.

Example:
find . -name "*.log" -size >10MB

## 9. Filter Evaluation Flow
- File discovered during traversal
- Metadata loaded
- Filters applied one by one
- If any filter fails → skip file
- If all filters pass → print path

This enables short-circuit evaluation and improves performance.

## 10. Handling Symbolic Links
- Ignore symbolic links in initial version
- Prevents infinite loops
- Advanced version can track visited inode numbers

## 11. Handling Permissions and Errors
- Catch permission denied errors
- Log error and continue traversal
- Do not terminate the entire search

## 12. Performance Optimizations
- Load file metadata lazily
- Skip directories early if filters cannot match
- Support depth limits (-maxdepth)
- Avoid unnecessary system calls

## 13. Command Interface

Example Commands:
find /home/user -name "*.java"
find /var/log -size +100MB
find . -type d

Parsed Into:
- Root directory
- List of filter objects

## 14. Time and Space Complexity
Time Complexity: O(N), where N is number of files and directories  
Space Complexity: O(D), where D is maximum directory depth

## 15. Trade-Offs
- DFS is memory efficient
- Iterative traversal avoids recursion limits
- Predicate filters make the system extensible
- No indexing ensures real-time accuracy but slower repeated searches

## 16. Future Enhancements
- Regex-based name matching
- OR conditions between filters
- Multi-threaded directory traversal
- File content search (grep integration)
- Indexing for faster repeated searches
