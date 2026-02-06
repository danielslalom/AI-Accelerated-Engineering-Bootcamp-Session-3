# Cloud Architecture Overview

## System Context

This document provides a high-level overview of the TODO App architecture, which is organized as a monorepo containing both frontend and backend components.

## Architecture Diagram

```mermaid
C4Context
    title System Context Diagram - TODO App

    Person(user, "User", "A person managing their tasks")
    
    System_Boundary(todoApp, "TODO App Monorepo") {
        Container(frontend, "React Frontend", "React, Material-UI", "Provides task management UI with filtering, priorities, and due dates")
        Container(backend, "Express API", "Node.js, Express", "REST API for task CRUD operations")
        ContainerDb(database, "In-Memory Store", "SQLite (better-sqlite3)", "Stores tasks in memory during runtime")
    }

    Rel(user, frontend, "Uses", "HTTPS")
    Rel(frontend, backend, "Makes API calls to", "HTTP/JSON")
    Rel(backend, database, "Reads from and writes to", "SQL")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
```

## Sequence Diagram - Creating a TODO

The following sequence diagram illustrates the interaction flow when a user creates a new TODO task:

```mermaid
sequenceDiagram
    actor User
    participant Browser as React Frontend
    participant API as Express API
    participant DB as SQLite (In-Memory)

    User->>Browser: Fill in task form (title, priority, due date)
    User->>Browser: Click "Add Task" button
    
    Browser->>Browser: Validate form input
    Note over Browser: Check title is not empty
    
    Browser->>API: POST /api/tasks
    Note over Browser,API: { title, description, due_date, priority }
    
    API->>API: Validate request body
    Note over API: Check title required<br/>Validate priority (P1/P2/P3)<br/>Validate date format
    
    API->>DB: INSERT INTO tasks
    Note over API,DB: INSERT INTO tasks (title, description, due_date, priority)<br/>VALUES (?, ?, ?, ?)
    
    DB-->>API: Return new task ID
    
    API->>DB: SELECT task by ID
    DB-->>API: Return complete task object
    
    API-->>Browser: 201 Created
    Note over API,Browser: { id, title, description, due_date, priority,<br/>completed, created_at }
    
    Browser->>Browser: Update task list state
    Browser->>Browser: Clear form inputs
    Browser->>User: Display new task in list
    
    Note over User,DB: Task successfully created and visible to user
```

## Components

### Frontend (`packages/frontend`)
- **Technology**: React 18, Material-UI (MUI)
- **Purpose**: Single-page application providing task management interface
- **Key Features**:
  - Task creation, editing, and deletion
  - Due date and priority assignment
  - Filter views (All, Today, Overdue)
  - Responsive UI with Material Design
- **Communication**: REST API calls to Express backend

### Backend (`packages/backend`)
- **Technology**: Node.js, Express, better-sqlite3
- **Purpose**: REST API server for task operations
- **Key Features**:
  - CRUD endpoints for tasks
  - Data validation
  - Query filtering and sorting
  - CORS support for local development
- **Communication**: HTTP/JSON REST API

### Data Store
- **Technology**: SQLite (in-memory)
- **Purpose**: Ephemeral task storage during application runtime
- **Characteristics**:
  - No persistence across server restarts
  - Suitable for development and demonstration
  - Fast read/write operations
  - SQL-based querying and sorting

## Data Flow

1. **User Interaction**: User interacts with React frontend in the browser
2. **API Request**: Frontend sends HTTP requests to Express backend (e.g., POST /api/tasks)
3. **Data Processing**: Backend validates and processes the request
4. **Database Operation**: Backend executes SQL queries against in-memory SQLite database
5. **Response**: Backend returns JSON response to frontend
6. **UI Update**: Frontend updates the view with the new data

## Deployment Considerations

### Current Setup (Development)
- Monorepo structure with shared workspace
- Frontend and backend run on separate ports
- In-memory database resets on server restart
- No persistence layer

### Future Considerations (Post-MVP)
- Replace in-memory SQLite with persistent database (PostgreSQL, MySQL)
- Deploy frontend as static site (Vercel, Netlify, S3)
- Deploy backend as containerized service (Docker, Kubernetes)
- Add environment-based configuration
- Implement proper authentication and authorization
- Add caching layer (Redis) for improved performance
- Set up CI/CD pipeline

## Technology Stack Summary

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Frontend | React 18 | UI framework |
| UI Components | Material-UI (MUI) | Component library |
| Backend | Express.js | Web server framework |
| Runtime | Node.js | JavaScript runtime |
| Database | SQLite (better-sqlite3) | In-memory data store |
| Package Manager | npm/yarn | Dependency management |
| Monorepo Structure | npm workspaces | Multi-package management |

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/tasks | List all tasks (with optional filtering) |
| POST | /api/tasks | Create a new task |
| GET | /api/tasks/:id | Get a specific task |
| PUT | /api/tasks/:id | Update a task |
| PATCH | /api/tasks/:id | Partial update (e.g., toggle completion) |
| DELETE | /api/tasks/:id | Delete a task |

## Security Considerations

### Current State
- No authentication or authorization
- CORS enabled for local development
- No input sanitization beyond basic validation
- No rate limiting

### Recommended Improvements
- Implement user authentication (JWT, OAuth)
- Add request validation middleware (express-validator)
- Implement rate limiting (express-rate-limit)
- Add input sanitization to prevent SQL injection
- Use HTTPS in production
- Implement CSRF protection
- Add logging and monitoring
