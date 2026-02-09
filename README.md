# Project Management System - Technical Documentation

**Documentation Created by:** Muhammed Nibras k                
**Email Id:** nibras.selecttraining@gmail.com

## Table of Contents

1. [Project Overview](#project-overview)
2. [Feature Breakdown](#feature-breakdown)
3. [System Architecture](#system-architecture)
4. [Frontend Flow](#frontend-flow)
5. [Backend Flow](#backend-flow)
6. [Database Design](#database-design)
7. [Real-Time & Notification System](#real-time--notification-system)
8. [Performance Optimizations](#performance-optimizations)
9. [Security Considerations](#security-considerations)
10. [Scalability & Maintainability](#scalability--maintainability)
11. [Typical User Journeys](#typical-user-journeys)

---

## Project Overview

### Problem Statement

Modern organizations struggle with fragmented project management tools that don't align with their organizational structure. Teams need a system that:

- Respects departmental boundaries while enabling cross-department collaboration
- Provides flexible permission models that match real-world workflows
- Supports hierarchical task structures (tasks with nested subtasks)
- Delivers real-time updates without constant page refreshes
- Scales from small teams to enterprise organizations

### Solution

A full-stack project management system built with **Laravel 12** (backend) and **Vue 3 + TypeScript** (frontend) that implements a **ClickUp-inspired permission model** combined with **department-based access control**. The system provides:

- **Department-scoped projects** with cross-department task assignments
- **Role-based access control** (Owner, Project Manager, Team Member, Collaborator)
- **Project Head model** where project creators have full control
- **Real-time collaboration** via WebSockets (Laravel Reverb)
- **Comprehensive notification system** with email and in-app alerts
- **Full-text search** across projects, tasks, and etc....
- **Public and collaborator sharing** for external stakeholders

### Target Users

1. **System Owners/Admins**: Full system access, user management, department oversight
2. **Project Managers**: Manage projects within their department, assign tasks
3. **Team Members**: Work on assigned tasks, create subtasks, collaborate
4. **Collaborators**: External users with limited access to shared projects
5. **External Viewers**: Public access via shareable links (read-only)

---

## Feature Breakdown

### User Roles & Permissions

#### 1. Owner (Admin)
- **Full system access**: View and manage all departments, projects, and users
- **User management**: Create, update, delete users, assign roles and departments
- **Department management**: Create, update, delete departments
- **Project access**: All projects across all departments
- **Task management**: Full CRUD on all tasks
- **Dashboard**: System-wide statistics and analytics

#### 2. Project Manager
- **Department scope**: Manage projects within their assigned department
- **Project creation**: Can create projects in their department
- **Task management**: Create, assign, and manage tasks in department projects
- **Cross-department visibility**: Can view projects where assigned to tasks (ClickUp model)
- **Dashboard**: Department-specific project statistics

#### 3. Team Member
- **Project access**: Only projects where:
  - Assigned to any task (ClickUp model)
  - Explicitly invited as a project member
  - Project creator (becomes Project Head)
  - Project is shared with them
- **Task operations**: 
  - View all tasks in accessible projects
  - Edit only assigned tasks
  - Create subtasks under assigned tasks
  - Comment on any accessible task
- **Dashboard**: Personal task summary and quick actions

#### 4. Collaborator
- **Limited access**: Only projects explicitly shared with them
- **Permissions**: Configurable (view, comment, edit) per share
- **Task operations**: Based on share permissions
- **No department access**: Cannot see department structure

#### 5. External Viewer (Public Shares)
- **Read-only access**: Via shareable token links
- **No authentication**: Public access without login
- **Limited features**: View project/task details only

### Core Workflows

#### Project Creation Workflow
1. **Any authenticated user** can create a project
2. Creator automatically becomes **Project Head** (`owner_id`)
3. Project is assigned to a department (user's department or selected)
4. A default **TaskList** is auto-created
5. Creator has full control: can invite members, assign roles, manage tasks

#### Task Assignment Workflow
1. **Project Head or PM** creates a root task
2. Task can be assigned to **any user** (cross-department allowed)
3. Assigned user receives:
   - Real-time notification (WebSocket)
   - Email notification (if enabled)
   - Access to project (if not already accessible)
4. Assigned user can:
   - Edit the task
   - Create subtasks
   - Comment and attach files

#### Subtask Creation Workflow
1. **Authorized users** can create subtasks under any task
2. Subtasks inherit project and task list from parent
3. Subtasks can have their own assignees (independent of parent)
4. Subtasks can be nested recursively (unlimited)
5. Parent task completion doesn't auto-complete subtasks (explicit action required)

#### Sharing Workflow
1. **Public Sharing**:
   - Project Head generates shareable token
   - Token provides read-only access without authentication
   - Token can be revoked or expired
   
2. **Collaborator Sharing**:
   - Project Head invites internal users
   - Permissions configured: view, comment, edit
   - Collaborator receives notification and access

---

## System Architecture

### Technology Stack

**Backend:**
- **Framework**: Laravel 12 (PHP 8.2+)
- **Authentication**: JWT (tymon/jwt-auth)
- **Real-time**: Laravel Reverb (WebSockets)
- **Queue**: Laravel Queue (database driver)
- **Database**: MySQL/MariaDB with FULLTEXT indexes
- **Caching**: Laravel Cache (file/database driver)

**Frontend:**
- **Framework**: Vue 3 (Composition API)
- **Language**: TypeScript
- **State Management**: Pinia
- **Routing**: Vue Router 4
- **HTTP Client**: Axios
- **Real-time**: Laravel Echo + Pusher JS
- **UI Framework**: Tailwind CSS 4
- **Build Tool**: Vite (rolldown-vite)

### Architecture Pattern

**Backend**: MVC with Service Layer
- **Controllers**: Handle HTTP requests, validation, authorization
- **Services**: Business logic (NotificationService, SearchService)
- **Models**: Eloquent ORM with relationships and scopes
- **Policies**: Authorization logic (TaskPolicy, ProjectPolicy)
- **Events**: Domain events for real-time broadcasting
- **Jobs**: Async processing (email notifications, cleanup)

**Frontend**: Component-Based SPA
- **Stores**: Centralized state (Pinia)
- **Composables**: Reusable logic (useRealTime, useNotifications)
- **Components**: Vue SFCs organized by feature
- **Views**: Page-level components
- **API Layer**: TypeScript interfaces and API clients

### System Components

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend (Vue 3)                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │   Views   │  │ Components│  │  Stores   │  │ Composables││
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│         │              │              │              │       │
│         └──────────────┴──────────────┴──────────────┘       │
│                            │                                  │
│                    ┌───────▼───────┐                          │
│                    │  Laravel Echo │                          │
│                    │  (WebSocket)  │                          │
│                    └───────┬───────┘                          │
└────────────────────────────┼──────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  Laravel Reverb │
                    │  (WebSocket)    │
                    └────────┬────────┘
                             │
┌────────────────────────────┼──────────────────────────────────┐
│                    Backend (Laravel 12)                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐     │
│  │Controllers│  │ Services │  │  Models  │  │ Policies │     │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘     │
│         │              │              │              │         │
│         └──────────────┴──────────────┴──────────────┘         │
│                            │                                    │
│                    ┌───────▼───────┐                           │
│                    │   Events &     │                           │
│                    │   Jobs Queue   │                           │
│                    └───────┬───────┘                           │
└────────────────────────────┼──────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │    MySQL DB     │
                    └─────────────────┘
```

---

## Frontend Flow

### Application Structure

```
frontend/src/
├── api/              # API client functions (TypeScript)
├── assets/           # Static assets
├── components/       # Reusable Vue components
│   ├── layout/      # AppLayout, Sidebar, Header
│   ├── tasks/       # TaskCard, TaskModal, SubtaskList
│   ├── projects/    # ProjectCard, ProjectForm
│   └── ...
├── composables/      # Reusable composition functions
│   ├── useRealTime.ts      # WebSocket subscriptions
│   ├── useNotifications.ts # Notification handling
│   └── ...
├── router/           # Vue Router configuration
├── stores/           # Pinia stores
│   ├── auth.store.ts
│   ├── task.store.ts
│   ├── project.store.ts
│   └── ...
├── views/            # Page-level components
└── utils/            # Helper functions
```

### State Management (Pinia Stores)

#### Auth Store (`auth.store.ts`)
- **State**: `user`, `token`, `isAuthenticated`
- **Actions**: `login()`, `logout()`, `register()`, `refreshToken()`
- **Getters**: `isOwner()`, `isProjectManager()`, `isTeamMember()`
- **Persistence**: Token stored in localStorage

#### Task Store (`task.store.ts`)
- **State**: 
  - `tasksByStatus`: Kanban columns (not_started, in_progress, review, completed)
  - `currentTask`: Selected task details
  - `comments`, `attachments`, `dependencies`: Ancillary data
- **Actions**:
  - `fetchByProject()`: Load all tasks for a project
  - `fetchById()`: Load single task with full tree
  - `createTask()`: Create new task (optimistic update)
  - `updateTask()`: Update task (optimistic update)
  - `deleteTask()`: Delete task and subtasks
- **Optimistic Updates**: UI updates immediately, syncs with server response

#### Project Store (`project.store.ts`)
- **State**: `projects`, `currentProject`, `loading`
- **Actions**: `fetchProjects()`, `createProject()`, `updateProject()`
- **Filtering**: By department, status, search query

#### Notification Store (`notification.store.ts`)
- **State**: `notifications[]`, `unreadCount`, `preferences`
- **Actions**: `fetchNotifications()`, `markAsRead()`, `dismiss()`
- **Real-time**: Listens to `notifications.{userId}` channel

### Component Responsibilities

#### AppLayout Component
- **Purpose**: Main application shell
- **Features**:
  - Sidebar navigation (role-based menu items)
  - Header with search, notifications, user menu
  - Router outlet for page content
- **Real-time Setup**: Initializes Echo connection on mount

#### TaskCard Component
- **Purpose**: Display task in Kanban/list view
- **Props**: `task`, `projectId`
- **Features**:
  - Drag & drop (SortableJS)
  - Status change on drop
  - Click to open task detail modal
  - Display assignees, priority, due date

#### TaskModal Component
- **Purpose**: Full task detail view
- **Features**:
  - Edit task fields (title, description, dates, priority)
  - Subtask management (create, edit, delete)
  - Comments section (real-time updates)
  - Attachments (upload, download, delete)
  - Dependencies (add/remove)
  - Activity timeline

### Real-Time Integration

#### WebSocket Connection Flow

1. **Initialization** (`main.ts`):
   ```typescript
   // On app start, initialize Echo if user is authenticated
   if (authStore.isAuthenticated) {
     initEcho(authStore.token)
   }
   ```

2. **Project Channel Subscription** (`useRealTime.ts`):
   ```typescript
   // Subscribe to project.{projectId} channel
   Echo.private(`project.${projectId}`)
     .listen('.TaskCreated', handleTaskCreated)
     .listen('.TaskUpdated', handleTaskUpdated)
     .listen('.TaskDeleted', handleTaskDeleted)
   ```

3. **Event Handlers**:
   - `handleTaskCreated`: Add task to appropriate Kanban column
   - `handleTaskUpdated`: Update task in store (merge changes)
   - `handleTaskDeleted`: Remove task from store
   - `handleCommentAdded`: Add comment to current task
   - `handleAttachmentAdded`: Add attachment to current task

4. **Optimistic Updates**:
   - User action → Immediate UI update → API call → Server response → Sync
   - If API fails, revert optimistic change and show error

### Routing & Navigation

#### Route Guards (`router/guards.ts`)
- **Auth Guard**: Checks JWT token, redirects to login if invalid
- **Role Guard**: Checks user role for protected routes (e.g., `/users` requires `owner`)
- **Guest Guard**: Redirects authenticated users away from login/register

#### Route Structure
- `/dashboard`: Role-based dashboard (Admin/PM/Team)
- `/projects`: List all accessible projects
- `/projects/:id`: Project detail (Kanban view)
- `/projects/:id/list`: Task list view
- `/my-tasks`: User's assigned tasks
- `/search`: Global search
- `/notifications`: Notification center
- `/settings`: User preferences

---

## Backend Flow

### Request Lifecycle

```
HTTP Request
    ↓
Middleware Stack
    ├── CORS
    ├── JWT Auth (jwt.auth)
    ├── Role Check (role:owner)
    └── ...
    ↓
Route Resolution (routes/api.php)
    ↓
Controller Method
    ├── Authorization Check ($this->authorize())
    ├── Validation (Validator::make())
    ├── Business Logic
    │   ├── Service Calls (NotificationService, SearchService)
    │   ├── Model Operations (Eloquent)
    │   └── Event Dispatch (TaskCreated, TaskUpdated)
    └── Response (JSON)
```

### API Structure (`routes/api.php`)

#### Route Organization
1. **Public Routes**: `/api/public-shares/{token}` (no auth)
2. **Auth Routes**: `/api/auth/*` (login, register, profile)
3. **Dashboard Routes**: Role-based dashboards
4. **Admin Routes**: `/api/admin/*` (owner only)
5. **Department Routes**: `/api/departments/*`
6. **Project Routes**: `/api/projects/*`
7. **Task Routes**: `/api/projects/{projectId}/tasks/*`
8. **User Routes**: `/api/users/*`
9. **Search Routes**: `/api/search/*`
10. **Notification Routes**: `/api/notifications/*`

### Controller Responsibilities

#### TaskController
- **index()**: Get all root tasks for a project (with nested subtasks)
- **store()**: Create new task (root or subtask)
- **show()**: Get single task with full tree
- **update()**: Update task fields
- **destroy()**: Soft delete task (cascades to subtasks)
- **updateStatus()**: Change task status (triggers notifications)
- **updatePosition()**: Drag & drop reordering

**Authorization Flow**:
```php
// Check if user can view project
$this->authorize('view', $project);

// Check if user can create task
$this->authorize('create', [Task::class, $project]);

// Check if user can update task
$this->authorize('update', $task);
```

#### ProjectController
- **index()**: List projects (filtered by user access)
- **store()**: Create project (any authenticated user)
- **show()**: Get project details
- **update()**: Update project (Project Head or PM)
- **destroy()**: Delete project (Project Head only)
- **getMembers()**: List project members
- **addMember()**: Invite user to project

**Access Control Logic**:
```php
// Team members see projects where:
// 1. Assigned to any task
// 2. Invited as member
// 3. Created by them
// 4. Shared with them
```

#### NotificationController
- **index()**: Paginated notifications
- **unreadCount()**: Get unread count
- **markAsRead()**: Mark single notification as read
- **markAllAsRead()**: Mark all as read
- **getPreferences()**: Get user notification preferences
- **updatePreferences()**: Update preferences

### Service Layer

#### NotificationService
**Purpose**: Centralized notification creation and management

**Key Methods**:
- `create()`: Create notification (checks preferences, broadcasts, queues email)
- `notifyTaskAssigned()`: Factory method for task assignment
- `notifyTaskUpdated()`: Factory method for task updates
- `notifyCommentAdded()`: Factory method for comments
- `getUserNotifications()`: Fetch with filters

**Flow**:
```
Event Triggered (e.g., TaskAssigned)
    ↓
NotificationService::notifyTaskAssigned()
    ↓
Check User Preferences (shouldShowInApp? shouldSendEmail?)
    ↓
Create Notification Record
    ↓
Broadcast Event (NotificationCreated) → WebSocket
    ↓
Queue Email Job (if enabled)
```

#### SearchService
**Purpose**: Full-text search across departments, projects, tasks, comments

**Features**:
- MySQL FULLTEXT indexes on searchable columns
- Role-based filtering (users only see accessible content)
- Query highlighting with `<mark>` tags
- Search history tracking
- Auto-suggestions (quick search)

**Search Flow**:
```
User Query: "dashboard redesign"
    ↓
SearchService::globalSearch()
    ↓
For each entity type:
    ├── Apply role-based filters
    ├── FULLTEXT search: MATCH(...) AGAINST(...)
    └── Highlight matches
    ↓
Return grouped results + search time
    ↓
Save to SearchHistory
```

### Event System

#### Events (`app/Events/`)
- **TaskCreated**: Broadcasts to `project.{projectId}` channel
- **TaskUpdated**: Broadcasts task changes
- **TaskDeleted**: Notifies deletion
- **TaskAssigned**: Notifies assignment
- **CommentAdded**: Broadcasts to `task.{taskId}` channel
- **AttachmentAdded**: Notifies file upload
- **NotificationCreated**: Broadcasts to `notifications.{userId}` channel

#### Broadcasting Channels (`routes/channels.php`)
- **Private Channels**: Require authentication
  - `project.{projectId}`: Project updates
  - `task.{taskId}`: Task-specific updates
  - `notifications.{userId}`: User notifications
- **Authorization**: Channel policies check user access

### Job Queue

#### Jobs (`app/Jobs/`)
- **SendEmailNotification**: Queued email sending
  - Retries: 3 attempts
  - Backoff: 60 seconds
  - Checks user preferences before sending
  
- **SendDigestEmail**: Daily digest of notifications
  - Scheduled via Laravel Scheduler
  
- **CleanOldNotifications**: Cleanup old read notifications
  - Scheduled daily

#### Queue Configuration
- **Driver**: Database (configurable to Redis/SQS)
- **Worker**: `php artisan queue:listen`
- **Failed Jobs**: Stored in `failed_jobs` table

### Observers

#### TaskObserver
- **Created**: Dispatch `TaskCreated` event
- **Updated**: Dispatch `TaskUpdated` event, create activity log
- **Deleted**: Dispatch `TaskDeleted` event

#### ProjectObserver
- **Created**: Create default TaskList, notify department members
- **Updated**: Create activity log
- **Deleted**: Cascade soft delete to tasks

---

## Database Design

### Core Tables

#### users
```sql
- id (PK)
- name
- email (unique)
- password (hashed)
- role (enum: owner, project_manager, team_member, collaborator, external_viewer)
- department_id (FK → departments.id, nullable)
- email_verified_at
- deleted_at (soft deletes)
- timestamps
```

**Indexes**:
- `email` (unique)
- `role, department_id` (composite)

#### departments
```sql
- id (PK)
- name (unique)
- description
- deleted_at
- timestamps
```

**Indexes**:
- `name` (unique)
- FULLTEXT: `name, description`

#### projects
```sql
- id (PK)
- department_id (FK → departments.id)
- name
- description
- owner_id (FK → users.id) -- Project Head
- start_date
- end_date
- status (enum: planning, active, on_hold, completed, cancelled)
- deleted_at
- timestamps
```

**Indexes**:
- `department_id`
- `owner_id`
- `status`
- FULLTEXT: `name, description`

#### tasks
```sql
- id (PK)
- project_id (FK → projects.id)
- task_list_id (FK → task_lists.id)
- parent_id (FK → tasks.id, nullable) -- For subtasks
- title
- description
- priority (enum: low, medium, high, urgent)
- status (enum: not_started, in_progress, review, completed)
- position (integer) -- For ordering
- start_date
- due_date
- created_by (FK → users.id)
- updated_by (FK → users.id, nullable)
- is_archived (boolean)
- deleted_at
- timestamps
```

**Indexes**:
- `project_id`
- `task_list_id, status, position` (composite)
- `parent_id`
- `created_by`
- FULLTEXT: `title, description`

#### task_user (Pivot)
```sql
- task_id (FK → tasks.id)
- user_id (FK → users.id)
- assigned_at
- assigned_by (FK → users.id)
- timestamps
```

**Indexes**:
- `task_id, user_id` (composite unique)
- `user_id`

#### project_user (Pivot)
```sql
- project_id (FK → projects.id)
- user_id (FK → users.id)
- role (enum: head, admin, manager, member, guest)
- joined_at
- timestamps
```

**Indexes**:
- `project_id, user_id` (composite unique)

#### project_shares (Collaborator Sharing)
```sql
- id (PK)
- project_id (FK → projects.id)
- user_id (FK → users.id)
- can_view (boolean)
- can_comment (boolean)
- can_edit (boolean)
- shared_by (FK → users.id)
- timestamps
```

#### public_shares (Public Sharing)
```sql
- id (PK)
- shareable_type (morph: Project/Task)
- shareable_id
- token (unique, indexed)
- expires_at (nullable)
- can_view (boolean)
- can_comment (boolean)
- can_edit (boolean)
- created_by (FK → users.id)
- timestamps
```

**Indexes**:
- `token` (unique)

#### notifications
```sql
- id (PK)
- user_id (FK → users.id)
- type (string) -- task_assigned, task_updated, comment_added, etc.
- notifiable_type (morph)
- notifiable_id
- title
- message
- data (JSON)
- priority (enum: low, normal, high, urgent)
- read_at
- dismissed_at
- timestamps
```

**Indexes**:
- `user_id, read_at`
- `user_id, created_at`
- `type`

#### task_comments
```sql
- id (PK)
- task_id (FK → tasks.id)
- user_id (FK → users.id)
- content
- deleted_at
- timestamps
```

**Indexes**:
- `task_id`
- FULLTEXT: `content`

#### task_attachments
```sql
- id (PK)
- task_id (FK → tasks.id)
- user_id (FK → users.id)
- filename
- original_filename
- mime_type
- size (bytes)
- path (storage path)
- timestamps
```

#### task_dependencies
```sql
- task_id (FK → tasks.id)
- depends_on_task_id (FK → tasks.id)
- timestamps
```

**Indexes**:
- `task_id, depends_on_task_id` (composite unique)

#### activity_logs
```sql
- id (PK)
- user_id (FK → users.id)
- loggable_type (morph)
- loggable_id
- action (string) -- created, updated, deleted, etc.
- description
- changes (JSON) -- before/after values
- timestamps
```

#### search_history
```sql
- id (PK)
- user_id (FK → users.id)
- query (string)
- search_type (string) -- global, projects, tasks, etc.
- results_count
- filters (JSON)
- searched_at
- timestamps
```

### Relationships

#### User Relationships
- `belongsTo(Department)`
- `hasMany(Project)` as owner
- `belongsToMany(Project)` via `project_user` (memberships)
- `belongsToMany(Task)` via `task_user` (assignments)
- `hasMany(Task)` as creator
- `hasMany(Notification)`

#### Project Relationships
- `belongsTo(Department)`
- `belongsTo(User)` as owner
- `hasOne(TaskList)`
- `hasMany(Task)`
- `belongsToMany(User)` via `project_user` (members)
- `belongsToMany(User)` via `project_shares` (collaborators)

#### Task Relationships
- `belongsTo(Project)`
- `belongsTo(TaskList)`
- `belongsTo(Task)` as parent
- `hasMany(Task)` as subtasks (recursive)
- `belongsToMany(User)` via `task_user` (assignees)
- `hasMany(TaskComment)`
- `hasMany(TaskAttachment)`
- `belongsToMany(Task)` via `task_dependencies` (dependencies)

### Indexing Strategy

#### Performance Indexes
- **Foreign Keys**: All FK columns indexed
- **Composite Indexes**: 
  - `task_list_id, status, position` (Kanban queries)
  - `user_id, read_at` (notification queries)
- **FULLTEXT Indexes**: 
  - `departments(name, description)`
  - `projects(name, description)`
  - `tasks(title, description)`
  - `task_comments(content)`

#### Query Optimization
- **Eager Loading**: Prevents N+1 queries
  ```php
  Task::with(['assignees', 'creator', 'allSubtasks'])->get()
  ```
- **Select Specific Columns**: Reduces memory usage
  ```php
  User::select('id', 'name', 'email')->get()
  ```
- **Scopes**: Reusable query filters
  ```php
  Task::rootTasks()->where('status', 'in_progress')->get()
  ```

---

## Real-Time & Notification System

### WebSocket Architecture

#### Laravel Reverb
- **Purpose**: WebSocket server for real-time communication
- **Protocol**: Pusher-compatible protocol
- **Channels**: Private channels with authentication
- **Connection**: Persistent WebSocket connection

#### Frontend Echo Setup
```typescript
// Initialize Echo with JWT token
initEcho(token)

// Subscribe to private channel
Echo.private(`project.${projectId}`)
  .listen('.TaskCreated', (data) => {
    // Handle task creation
  })
```

### Broadcasting Channels

#### Project Channel (`project.{projectId}`)
**Purpose**: Broadcast project-wide updates

**Events**:
- `TaskCreated`: New task added
- `TaskUpdated`: Task modified
- `TaskDeleted`: Task removed
- `TaskAssigned`: User assigned to task
- `AttachmentAdded`: File uploaded
- `AttachmentDeleted`: File removed

**Authorization**: User must have access to project (`canViewProject()`)

#### Task Channel (`task.{taskId}`)
**Purpose**: Broadcast task-specific updates

**Events**:
- `CommentAdded`: New comment
- `TaskUpdated`: Task changes
- `AttachmentAdded`: File uploaded

**Authorization**: User must have access to task's project

#### Notification Channel (`notifications.{userId}`)
**Purpose**: User-specific notifications

**Events**:
- `notification.created`: New notification
- `notification.read`: Notification marked as read
- `notification.dismissed`: Notification dismissed

**Authorization**: User ID must match authenticated user

### Notification System

#### Notification Types
- **Task Events**: `task_assigned`, `task_updated`, `task_completed`, `task_deleted`
- **Comment Events**: `comment_added`, `mention`
- **Project Events**: `project_created`, `project_updated`, `project_shared`, `project_member_added`
- **Reminders**: `due_date_reminder`, `overdue`

#### Notification Flow

```
Action Triggered (e.g., Task Assigned)
    ↓
NotificationService::notifyTaskAssigned()
    ↓
Check User Preferences
    ├── In-App: Enabled? → Create Notification Record
    └── Email: Enabled? → Queue Email Job
    ↓
Broadcast NotificationCreated Event
    ↓
WebSocket → Frontend (Real-time)
    ↓
Email Job → SMTP Server (Async)
```

#### Notification Preferences
- **Per-Type Control**: Enable/disable each notification type
- **Channels**: In-app, email, or both
- **Sound**: Enable/disable sound alerts
- **Defaults**: Sensible defaults for each role

#### Email Notifications
- **Template System**: Customizable email templates
- **Queue Processing**: Async email sending
- **Retry Logic**: 3 attempts with 60s backoff
- **Failure Handling**: Logged to `failed_jobs` table

### Edge Cases & Error Handling

#### WebSocket Disconnection
- **Auto-Reconnect**: Pusher JS handles reconnection
- **State Sync**: On reconnect, fetch latest data from API
- **Connection Status**: UI indicator for connection state

#### Notification Delivery Failures
- **Email Failures**: Retry 3 times, then log to failed_jobs
- **WebSocket Failures**: Logged, but don't block user action
- **Duplicate Prevention**: Idempotent notification creation

#### Concurrent Updates
- **Optimistic Updates**: Frontend updates immediately
- **Conflict Resolution**: Server response overwrites optimistic update
- **Last-Write-Wins**: Simple conflict resolution (can be enhanced with versioning)

---

## Performance Optimizations

### Backend Optimizations

#### Database Query Optimization
1. **Eager Loading**: Prevent N+1 queries
   ```php
   Task::with(['assignees', 'creator', 'allSubtasks'])->get()
   ```

2. **Select Specific Columns**: Reduce memory usage
   ```php
   User::select('id', 'name', 'email')->get()
   ```

3. **Query Scopes**: Reusable filters
   ```php
   Task::rootTasks()->where('status', 'in_progress')->get()
   ```

4. **Indexes**: Strategic indexes on frequently queried columns
   - Foreign keys
   - Status/priority columns
   - Composite indexes for common queries

#### Caching Strategy
- **Configuration Cache**: `php artisan config:cache`
- **Route Cache**: `php artisan route:cache`
- **View Cache**: Blade templates compiled
- **Query Results**: Not currently cached (can be added for dashboards)

#### Queue Processing
- **Async Operations**: Email sending, cleanup jobs
- **Batch Processing**: Multiple notifications batched
- **Job Prioritization**: High-priority notifications processed first

### Frontend Optimizations

#### Code Splitting
- **Route-Based**: Each route loads its own bundle
- **Component Lazy Loading**: Large components loaded on demand
- **Vite Optimization**: Tree-shaking, minification

#### State Management
- **Optimistic Updates**: Immediate UI feedback
- **Selective Updates**: Only update changed data
- **Memoization**: Computed properties cached

#### API Request Optimization
- **Request Debouncing**: Search queries debounced
- **Request Cancellation**: Cancel in-flight requests on navigation
- **Pagination**: Large lists paginated

#### Real-Time Optimization
- **Channel Subscription Management**: Subscribe only when needed
- **Event Filtering**: Filter events on frontend
- **Batch Updates**: Group multiple updates

### Search Performance

#### FULLTEXT Indexes
- **MySQL FULLTEXT**: Fast text search
- **Boolean Mode**: Supports complex queries
- **Minimum Word Length**: 3 characters (configurable)

#### Search Query Optimization
- **Role-Based Filtering**: Reduces search space
- **Result Limits**: Max 30 tasks, 20 projects, 20 comments
- **Search Time Tracking**: Monitor performance

---

## Security Considerations

### Authentication

#### JWT Authentication
- **Library**: tymon/jwt-auth
- **Token Storage**: localStorage (frontend)
- **Token Expiry**: Configurable (default: 60 minutes)
- **Refresh Token**: Supported via `/api/auth/refresh`
- **Token Revocation**: Logout invalidates token

#### Password Security
- **Hashing**: bcrypt (Laravel default)
- **Validation**: Minimum 8 characters, complexity rules
- **Reset Flow**: Email-based password reset

### Authorization

#### Policy-Based Authorization
- **TaskPolicy**: Controls task access
- **ProjectPolicy**: Controls project access
- **DepartmentPolicy**: Controls department access

#### Role-Based Access Control (RBAC)
- **Middleware**: `role:owner` middleware for route protection
- **Model Methods**: `isOwner()`, `isProjectManager()`, etc.
- **Permission Checks**: `canViewProject()`, `canEditTask()`, etc.

#### ClickUp Model Permissions
- **Project Head**: Full control over their projects
- **Cross-Department Access**: Users can access projects via task assignments
- **Granular Permissions**: View, comment, edit per share

### Data Protection

#### Input Validation
- **Laravel Validator**: All inputs validated
- **Sanitization**: XSS protection via Blade templating
- **SQL Injection**: Prevented via Eloquent ORM

#### File Upload Security
- **Validation**: File type, size limits
- **Storage**: Private storage for attachments
- **Download Authorization**: Check user access before download

#### Soft Deletes
- **Data Retention**: Deleted records retained for recovery
- **Cascade Deletes**: Subtasks soft-deleted with parent
- **Restore Functionality**: Users can restore deleted items

### API Security

#### CORS Configuration
- **Allowed Origins**: Configured in `config/cors.php`
- **Credentials**: JWT tokens sent with requests

#### Rate Limiting
- **Laravel Throttle**: Configurable rate limits
- **API Routes**: Throttled per user/IP

#### HTTPS
- **Production**: HTTPS required
- **WebSocket**: WSS for secure connections

### Sharing Security

#### Public Shares
- **Token Generation**: Cryptographically secure random tokens
- **Expiration**: Optional expiration dates
- **Revocation**: Tokens can be revoked
- **Read-Only**: Public shares are read-only by default

#### Collaborator Shares
- **Permission Levels**: View, comment, edit
- **Access Control**: Checked on every request
- **Audit Trail**: Activity logs track access

---

## Scalability & Maintainability

### Scalability Considerations

#### Database Scaling
- **Read Replicas**: Can add MySQL read replicas
- **Partitioning**: Large tables can be partitioned by date
- **Indexing**: Strategic indexes support growth

#### Application Scaling
- **Horizontal Scaling**: Stateless Laravel app scales horizontally
- **Load Balancing**: Multiple app servers behind load balancer
- **Session Storage**: Database sessions (can move to Redis)

#### Queue Scaling
- **Queue Workers**: Multiple workers process jobs
- **Queue Drivers**: Can switch to Redis/SQS for better performance
- **Job Prioritization**: High-priority jobs processed first

#### WebSocket Scaling
- **Reverb Scaling**: Can run multiple Reverb servers
- **Redis Pub/Sub**: Can use Redis for multi-server broadcasting
- **Connection Limits**: Monitor WebSocket connections

### Code Organization

#### Backend Structure
```
app/
├── Console/Commands/     # Artisan commands
├── Events/              # Domain events
├── Http/
│   ├── Controllers/    # API controllers
│   └── Middleware/     # Custom middleware
├── Jobs/               # Queue jobs
├── Mail/               # Email classes
├── Models/             # Eloquent models
├── Observers/          # Model observers
├── Policies/           # Authorization policies
├── Providers/          # Service providers
└── Services/           # Business logic services
```

#### Frontend Structure
```
src/
├── api/                # API clients
├── components/         # Vue components
├── composables/        # Composition functions
├── router/             # Vue Router
├── stores/             # Pinia stores
├── views/              # Page components
└── utils/              # Helper functions
```

### Testing Strategy

#### Backend Tests
- **Unit Tests**: Model methods, services
- **Feature Tests**: API endpoints, authorization
- **Test Coverage**: Aim for 80%+ coverage

#### Frontend Tests
- **Component Tests**: Vue component testing
- **E2E Tests**: Critical user flows
- **Integration Tests**: API integration

### Monitoring & Logging

#### Logging
- **Laravel Log**: File-based logging
- **Error Tracking**: Exceptions logged
- **Activity Logs**: User actions tracked

#### Performance Monitoring
- **Query Logging**: Slow queries logged
- **Response Times**: API response times tracked
- **WebSocket Metrics**: Connection counts monitored

### Deployment Considerations

#### Environment Configuration
- **.env File**: Environment-specific config
- **Config Caching**: Production config cached
- **Asset Compilation**: Frontend assets built for production

#### Database Migrations
- **Migration Strategy**: Run migrations on deployment
- **Rollback Plan**: Migrations can be rolled back
- **Seeders**: Optional seeders for development

#### CI/CD Pipeline
- **Automated Testing**: Run tests before deployment
- **Build Process**: Compile frontend assets
- **Deployment Scripts**: Automated deployment scripts

---

## Typical User Journeys

### Journey 1: Project Manager Creates Project and Assigns Tasks

**Actor**: Project Manager (Sarah, Engineering Department)

**Steps**:
1. **Login**: Sarah logs in with her credentials
2. **Navigate to Projects**: Clicks "Projects" in sidebar
3. **Create Project**: 
   - Clicks "New Project"
   - Enters: Name: "Q4 Dashboard Redesign", Description: "Redesign admin dashboard"
   - Selects: Department: Engineering
   - Clicks "Create"
   - **Backend**: Creates project, sets `owner_id` to Sarah, creates default TaskList
   - **Event**: `ProjectCreated` broadcasted to department
4. **Add Team Members**:
   - Opens project settings
   - Invites: John (Design), Mike (Frontend), Lisa (Backend)
   - Sets roles: John (member), Mike (member), Lisa (admin)
   - **Backend**: Creates `project_user` records, sends notifications
5. **Create Root Tasks**:
   - Opens Kanban view
   - Creates task: "Design Mockups" (status: not_started)
   - Assigns to: John
   - **Backend**: Creates task, assigns to John, broadcasts `TaskCreated`, sends notification
6. **Create Subtasks**:
   - Opens "Design Mockups" task
   - Creates subtask: "Homepage Mockup"
   - Creates subtask: "Dashboard Mockup"
   - **Backend**: Creates subtasks with `parent_id` set
7. **Monitor Progress**:
   - Views dashboard: Sees project statistics
   - Receives real-time updates: Task status changes appear instantly

**Technical Flow**:
```
POST /api/projects
  → ProjectController::store()
  → Creates Project (owner_id = Sarah)
  → ProjectObserver::created() → Creates TaskList
  → Broadcasts ProjectCreated event
  → Returns project data

POST /api/projects/{id}/members
  → ProjectController::addMember()
  → Creates project_user record
  → NotificationService::notifyProjectMemberAdded()
  → Broadcasts notification

POST /api/projects/{id}/tasks
  → TaskController::store()
  → Creates Task
  → TaskObserver::created() → Broadcasts TaskCreated
  → NotificationService::notifyTaskAssigned()
  → Returns task data
```

### Journey 2: Team Member Works on Assigned Task

**Actor**: Team Member (Mike, Frontend Developer)

**Steps**:
1. **Login**: Mike logs in
2. **View My Tasks**: Clicks "My Tasks" in sidebar
   - **Backend**: Queries tasks where Mike is assigned (`task_user` join)
   - Sees: "Design Mockups" task assigned to him
3. **Open Task**:
   - Clicks task card
   - **Backend**: Fetches task with full tree (`loadFullData()`)
   - Sees: Parent task + 2 subtasks
4. **Update Task Status**:
   - Drags task to "In Progress" column
   - **Backend**: Updates `status`, broadcasts `TaskUpdated`
   - **Real-time**: Other users see status change instantly
5. **Add Comment**:
   - Types: "Starting work on homepage mockup"
   - Clicks "Post Comment"
   - **Backend**: Creates `TaskComment`, broadcasts `CommentAdded`
   - **Notification**: John (task assignee) receives notification
6. **Upload Attachment**:
   - Uploads: "homepage-wireframe.pdf"
   - **Backend**: Stores file, creates `TaskAttachment`, broadcasts `AttachmentAdded`
7. **Create Subtask**:
   - Creates: "Responsive Design Check"
   - **Backend**: Creates subtask with `parent_id` set
   - **Authorization**: Allowed because Mike is assigned to parent task

**Technical Flow**:
```
GET /api/users/{userId}/tasks
  → TaskController::getUserTasks()
  → Queries tasks with assignees join
  → Returns tasks

PATCH /api/tasks/{id}/status
  → TaskController::updateStatus()
  → Updates task status
  → TaskObserver::updated() → Broadcasts TaskUpdated
  → NotificationService::notifyTaskUpdated()
  → Returns updated task

POST /api/tasks/{id}/comments
  → TaskCommentController::store()
  → Creates comment
  → Broadcasts CommentAdded
  → NotificationService::notifyCommentAdded()
  → Returns comment
```

### Journey 3: Owner Manages System-Wide Resources

**Actor**: System Owner (Admin)

**Steps**:
1. **Login**: Admin logs in
2. **View Admin Dashboard**:
   - Sees: Total departments, projects, tasks, users
   - **Backend**: Aggregates across all departments
3. **Create Department**:
   - Creates: "Marketing" department
   - **Backend**: Creates department, only owners can do this
4. **Create User**:
   - Creates: New user "Alice" with role "project_manager"
   - Assigns: Department "Marketing"
   - **Backend**: Creates user, sends welcome email
5. **View All Projects**:
   - Sees: Projects from all departments
   - **Backend**: No filtering (owner sees all)
6. **Manage User**:
   - Updates: Alice's role to "team_member"
   - **Backend**: Updates user, logs activity

**Technical Flow**:
```
GET /api/admin/dashboard
  → AdminDashboardController::index()
  → Aggregates: Department::count(), Project::count(), Task::count()
  → Returns statistics

POST /api/admin/users
  → UserManagementController::createUser()
  → Creates user with role and department
  → Sends welcome email
  → Returns user

PUT /api/admin/users/{id}
  → UserManagementController::updateUser()
  → Updates user
  → Logs activity
  → Returns updated user
```

### Journey 4: Collaborator Accesses Shared Project

**Actor**: Collaborator (External Stakeholder)

**Steps**:
1. **Receives Invitation**: Email notification with project share link
2. **Login**: Logs in (or creates account)
3. **View Shared Projects**:
   - Navigates to "Shared Projects"
   - **Backend**: Queries `project_shares` where `user_id` = collaborator
   - Sees: "Q4 Dashboard Redesign" project
4. **View Project**:
   - Opens project (read-only based on permissions)
   - **Backend**: Checks `project_shares` for permissions
   - Can view tasks but cannot edit (based on share settings)
5. **Add Comment**:
   - Comments on task: "Looks good, but consider adding dark mode"
   - **Backend**: Checks `can_comment` permission
   - Creates comment, broadcasts to project channel
6. **Receive Updates**:
   - Real-time: Sees task status changes via WebSocket
   - **Backend**: Subscribed to `project.{projectId}` channel

**Technical Flow**:
```
GET /api/collaborator-shares/my-projects
  → CollaboratorShareController::getMySharedProjects()
  → Queries project_shares where user_id = collaborator
  → Returns shared projects

GET /api/collaborator-shares/projects/{id}
  → CollaboratorShareController::viewSharedProject()
  → Checks project_shares for permissions
  → Returns project data

POST /api/collaborator-shares/tasks/{id}/comments
  → CollaboratorShareController::addCommentToSharedTask()
  → Checks can_comment permission
  → Creates comment
  → Broadcasts CommentAdded
```

### Journey 5: Public Share Access (No Authentication)

**Actor**: External Viewer (No Account)

**Steps**:
1. **Receives Link**: Gets public share link via email/Slack
2. **Opens Link**: `/public/projects/{token}`
   - **Backend**: Validates token, checks expiration
   - **No Auth Required**: Public route
3. **Views Project**:
   - Sees: Project details, tasks, comments (read-only)
   - **Backend**: Fetches project data, filters based on share permissions
4. **Cannot Edit**: All edit actions disabled (read-only)

**Technical Flow**:
```
GET /api/public-shares/{token}
  → PublicShareController::accessPublicShare()
  → Validates token, checks expiration
  → Returns project data (read-only)
  → No authentication required
```
### Journey 6: Team Member Creates Project and Assigns Cross-Department Tasks

**Actor**: Team Member (Alex, Engineering Department)

**Steps**:
1. **Login**: Alex logs in with team_member role
2. **Create Project**:
   - Navigates to "Projects" → Clicks "New Project"
   - Enters: Name: "Mobile App Integration", Description: "Integrate mobile app with backend API"
   - Selects: Department: Engineering (Alex's department)
   - Clicks "Create"
   - **Backend**: 
     - Creates project with `owner_id` = Alex (automatically becomes Project Head)
     - Creates default TaskList
     - Sets `department_id` = Engineering
     - **Key Point**: Alex now has full control over this project (Project Head privileges)
   - **Event**: `ProjectCreated` broadcasted to Engineering department
3. **Verify Project Head Status**:
   - Opens project settings
   - Sees: "Project Head" badge/indicator
   - **Backend**: `Project::isHead(Alex)` returns `true` (because `owner_id` = Alex)
   - Can now: Invite members, assign roles, manage all tasks, delete project
4. **Invite Cross-Department Members**:
   - Opens "Project Members" section
   - Invites: Sarah (Design Department), Mike (Marketing Department)
   - Sets roles: Sarah (member), Mike (member)
   - **Backend**: Creates `project_user` records
   - **Notification**: Sarah and Mike receive "Added to project" notifications
   - **Access**: Sarah and Mike can now view the project (cross-department access)
5. **Create Root Task**:
   - Opens Kanban view
   - Creates task: "Design Mobile UI Mockups"
   - Assigns to: Sarah (Design Department - different department)
   - **Backend**: 
     - Creates task in project
     - Creates `task_user` record linking task to Sarah
     - **Key Point**: Cross-department assignment is allowed (ClickUp model)
     - Sarah automatically gains access to project (via task assignment)
   - **Event**: `TaskCreated` and `TaskAssigned` broadcasted
   - **Notification**: Sarah receives task assignment notification
6. **Create Another Cross-Department Task**:
   - Creates task: "Marketing Campaign Materials"
   - Assigns to: Mike (Marketing Department)
   - **Backend**: Same flow - cross-department assignment
   - **Access**: Mike gains project access via task assignment
7. **Verify Full Control**:
   - Alex (Project Head) can:
     - Edit any task (even those assigned to others)
     - Delete any task
     - Change task priorities and statuses
     - Manage all project members
     - Delete the entire project
   - **Backend**: `TaskPolicy::update()` and `ProjectPolicy::update()` check `isProjectHead()`
8. **Monitor Cross-Department Collaboration**:
   - Views project dashboard
   - Sees: Tasks from multiple departments (Engineering, Design, Marketing)
   - **Real-time**: Receives updates when Sarah or Mike update their tasks
   - **Backend**: All users subscribed to `project.{projectId}` channel

**Technical Flow**:
```
POST /api/projects
  → ProjectController::store()
  → Validates: Any authenticated user can create (ProjectPolicy::create() = true)
  → Creates Project:
     - owner_id = Alex (current user)
     - department_id = Engineering (Alex's department)
  → ProjectObserver::created() → Creates default TaskList
  → Broadcasts ProjectCreated event to project.{projectId}
  → Returns project data
  → **Result**: Alex is now Project Head with full control

POST /api/projects/{id}/members
  → ProjectController::addMember()
  → Authorization: Checks if Alex can manage members (Project Head = true)
  → Creates project_user records:
     - Sarah (Design Department)
     - Mike (Marketing Department)
  → NotificationService::notifyProjectMemberAdded()
  → Broadcasts notifications
  → **Result**: Cross-department members added

POST /api/projects/{id}/tasks
  → TaskController::store()
  → Authorization: Project Head can create tasks (TaskPolicy::create() = true)
  → Creates Task:
     - project_id = project.id
     - assignees: [Sarah] (from Design Department)
  → TaskObserver::created() → Broadcasts TaskCreated
  → Creates task_user records (cross-department assignment)
  → NotificationService::notifyTaskAssigned()
  → **Key Point**: Sarah (Design) can now access Engineering project
  → **Backend**: User::canViewProject() checks task assignments
  → Returns task data

GET /api/projects/{id}/assignable-users
  → ProjectController::getAssignableUsers()
  → Returns: All users across all departments (ClickUp model)
  → **Key Point**: Not limited to project's department
  → Alex can assign tasks to anyone in the system

PATCH /api/tasks/{id}
  → TaskController::update()
  → Authorization: Project Head can update any task (TaskPolicy::update() = true)
  → Updates task
  → TaskObserver::updated() → Broadcasts TaskUpdated
  → **Result**: Alex has full control over all tasks
```

**Key Features Demonstrated**:
-  **Any user can create projects** (not just Project Managers)
-  **Creator automatically becomes Project Head** (`owner_id` = creator)
-  **Project Head has full control** (can manage all tasks, members, settings)
-  **Cross-department task assignments** (ClickUp model)
-  **Automatic project access** (users gain access when assigned to tasks)
-  **Real-time collaboration** across departments

---

## Conclusion

This project management system provides a robust, scalable solution for team collaboration with:

- **Flexible Permission Model**: ClickUp-inspired with department boundaries
- **Real-Time Collaboration**: WebSocket-based updates
- **Comprehensive Notifications**: In-app and email notifications
- **Full-Text Search**: Fast, role-filtered search
- **Scalable Architecture**: Laravel + Vue 3, ready for growth
- **Security**: JWT auth, policy-based authorization, data protection

The system is designed to handle growth from small teams to enterprise organizations while maintaining performance and security.
