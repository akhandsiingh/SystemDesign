# Spreadsheet System Design

## 1. Problem Statement
Design a spreadsheet application that supports a grid of cells, allows
users to enter values and formulas, automatically recalculates dependent
cells, and provides fast updates when data changes.

## 2. Assumptions
- Single user in the initial version
- Spreadsheet size is moderate (e.g., 10k × 100 cells)
- Cells can store numbers, text, or formulas
- Formulas can reference other cells (A1, B2, etc.)
- No macros or scripting in the first version
- Autosave is enabled
- Single timezone

## 3. Key Requirements

Functional Requirements
- Create spreadsheet
- Edit cell values
- Enter formulas (e.g., =A1+B1)
- Recalculate dependent cells automatically
- Display errors (e.g., #DIV/0, #CYCLE)

Non-Functional Requirements
- Low latency recalculation
- Correct dependency handling
- Scales to large sheets
- Crash-safe autosave

## 4. High-Level Architecture
User Input → Spreadsheet Engine → Formula Parser  
→ Dependency Graph → Recalculation Engine → UI Update

## 5. Relevant Tech Stack
Frontend: React  
Core Engine: Java  
Backend (optional): Java Spring Boot  
Database: PostgreSQL or MongoDB  
Cache (optional): Redis  
Deployment: AWS EC2  

For a local spreadsheet application, the backend can be skipped.

## 6. Core Components

Cell
- cellId (A1, B2, etc.)
- value
- formula
- computedValue
- errorState

Sheet
- sheetId
- collection of cells

Formula Engine
- Parses formulas
- Evaluates expressions

Dependency Graph
- Tracks relationships between cells

## 7. Data Model (Short)

Cell
- cell_id (VARCHAR)
- value (STRING)
- formula (STRING)
- computed_value (DECIMAL)
- error (VARCHAR)

Dependency
- from_cell (VARCHAR)
- to_cell (VARCHAR)

Meaning: to_cell depends on from_cell

## 8. Formula Parsing and Evaluation

Example:
C1 = A1 + B1

Steps:
- Parse the formula
- Extract dependencies (A1, B1)
- Store dependency edges
- Evaluate the expression
- Store computed value

## 9. Dependency Graph

Purpose:
- Automatically update dependent cells when a value changes

Representation:
- Directed Acyclic Graph (DAG)

Example:
A1 → C1  
B1 → C1  

Recalculation Algorithm:
- Detect changed cell
- Traverse all dependent cells
- Recalculate in topological order

This avoids recalculating the entire sheet.

## 10. Cycle Detection

Problem:
- Circular references cause infinite recalculation

Example:
A1 = B1 + 1  
B1 = A1 + 1  

Solution:
- Detect cycles while building the dependency graph
- Reject formula and mark cell with #CYCLE error

## 11. Recalculation Flow
- User edits a cell
- Update cell value or formula
- Identify dependent cells from dependency graph
- Recalculate affected cells only
- Update the UI

## 12. Performance Optimization
- Recalculate only impacted cells
- Cache computed values
- Batch multiple updates
- Lazy evaluation for off-screen cells

## 13. API Design (If Backend Exists)
POST /sheet/create  
PUT /cell/update  
GET /sheet/{id}  
POST /autosave  

## 14. Failure Handling
- Invalid formula → error state
- Division by zero → #DIV/0
- Circular dependency → reject formula
- Crash → restore from autosave

## 15. Scalability Considerations
- Partition sheets by sheetId
- Cache frequently accessed sheets
- Use WebWorkers for heavy calculations
- Move calculation engine to backend for very large sheets

## 16. Trade-Offs
- Dependency graph ensures correctness but uses extra memory
- Incremental recalculation improves performance
- No macros keeps system simple
- Single-user focus reduces complexity

## 17. Future Enhancements
- Real-time collaboration using WebSocket
- CRDT or OT for multi-user editing
- Charts and graphs
- Pivot tables
- Macros and scripting
- Excel import and export
