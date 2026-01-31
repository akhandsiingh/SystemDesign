# Text Editor with Cursor Operations System Design

## 1. Problem Statement
Design a simplified text editor that supports inserting characters,
deleting characters, moving the cursor, and undo/redo operations while
remaining fast, memory-efficient, and easy to reason about.

## 2. Assumptions
- Single user (no collaboration)
- Plain text only (no formatting)
- Single cursor (no selection)
- Document size can be large (10^5–10^6 characters)
- Operations should be O(1) or near O(1)
- Runs locally (no backend)

## 3. Key Requirements

Functional Requirements
- Insert character at cursor
- Delete character (backspace / delete)
- Move cursor left and right
- Move cursor up and down (line-based)
- Undo last operation

Non-Functional Requirements
- Low latency typing
- Efficient memory usage
- Scales to large documents

## 4. High-Level Architecture
Keyboard Input → Editor Core → Data Structure  
→ Cursor Update → Screen Render

## 5. Relevant Tech Stack
Language: Java or C++  
Core Data Structure: Two Stacks (Cursor-based editor)  
UI (optional): Console or simple GUI  

No database or backend required.

## 6. Core Design Decision

Avoid using String or ArrayList<Character> because insert and delete
operations are O(n).

Use a cursor-based data structure that supports O(1) operations.

## 7. Chosen Data Structure: Two Stack Approach

Left Stack:
- Characters before the cursor

Right Stack:
- Characters after the cursor

Example:

Text: HEL|LO  
Left Stack: H E L  
Right Stack: L O  

This enables constant-time cursor and edit operations.

## 8. Core Operations

Insert Character:
- Push character onto left stack

Backspace:
- Pop character from left stack

Delete:
- Pop character from right stack

Move Cursor Left:
- Pop from left stack and push to right stack

Move Cursor Right:
- Pop from right stack and push to left stack

All operations are O(1).

## 9. Cursor Up and Down (Line-Based)

Approach:
- Track newline characters ('\n')
- While moving up or down:
  - Count characters until previous or next newline
  - Adjust stacks accordingly

This approach is acceptable for fresher-level systems and can be optimized later.

## 10. Undo and Redo

Approach:
- Maintain an operation stack
- Each operation records enough data to reverse it

Examples:
- INSERT → undo by DELETE
- DELETE → undo by INSERT

This provides simple and efficient undo/redo support.

## 11. Sample Pseudo Code

Stack<Character> left = new Stack<>();
Stack<Character> right = new Stack<>();

void insert(char c) {
    left.push(c);
}

void moveLeft() {
    if (!left.isEmpty())
        right.push(left.pop());
}

void moveRight() {
    if (!right.isEmpty())
        left.push(right.pop());
}

void backspace() {
    if (!left.isEmpty())
        left.pop();
}

## 12. Rendering Text

To render the full text:
- Combine left stack
- Reverse right stack and append

Rendering cost is O(n) and acceptable since it is not done per keystroke.

## 13. Time Complexity Summary

Operation        Time
Insert           O(1)
Delete           O(1)
Cursor Move      O(1)
Undo / Redo      O(1)
Render           O(n)

## 14. Alternative Data Structures

Gap Buffer:
- Used by Emacs
- Efficient near cursor

Rope:
- Used for very large documents
- Efficient splits and merges

Piece Table:
- Used by VS Code
- Efficient undo history

For fresher-level design, the two-stack approach is the simplest and clearest.

## 15. Trade-Offs
- Two stacks provide simplicity and speed
- Cursor up/down adds some complexity
- Rendering cost is linear
- No formatting reduces system complexity

## 16. Future Enhancements
- Text selection
- Copy and paste
- Syntax highlighting
- Large file optimization using Rope
- Collaboration support
