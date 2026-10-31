#Top 5 Findings(UI and non functional)
### 1. Create task in any column(UI)
**Severity:** Critical

When a user adds a task and picks In progress, In review, or Done in the status dropdown, the app creates the card in that column, but it should only allow creating tasks in To do.

### 2. Task update has no membership or role check
**File:** `src/app/api/tasks/[id]/route.ts` (lines 16–37)
**Category:** Security
**Severity:** Critical

When any logged-in user sends `PATCH /api/tasks/:id`, the app updates that task after only checking a valid JWT. It never calls `getProjectMembership` or `canEditTasks`, unlike create and delete. Viewers and users from other projects can change title, status, and assignee if they know a task id, so board access control does not hold.

### 3. Task search builds SQL from request strings
**File:** `src/app/api/projects/[id]/tasks/route.ts` (lines 23–35)
**Category:** Security
**Severity:** Critical

When a client calls `GET /api/projects/:id/tasks?q=…`, `projectId` and `q` are concatenated into `$queryRawUnsafe`. Search should use bound parameters or Prisma `contains` filters. Untrusted input in SQL can read or change task rows outside the caller’s project.

### 4. Project detail returns password hashes
**File:** `src/app/api/projects/[id]/route.ts` (lines 25–44)
**Category:** Security
**Severity:** Critical

When a member loads a project (the board page’s `GET /api/projects/:id`), Prisma includes full `User` records for owner, members, assignees, and creators. Those objects include `passwordHash`. The API should select only `id`, `name`, and `email`. Hashes in the browser payload let anyone on the project attempt offline password cracking.

# UI findings (prioritized by business impact)

### 1. Skip workflow stages
**Severity:** High

When a user opens a task and changes status from To do directly to Done (or any non-next stage) and clicks save, the app moves the card to that column, but it should only allow moving to the next stage (To do → In progress → In review → Done).

### 2. Unassigned work past To do
**Severity:** High

When a user moves a task out of To do while Assignee is Unassigned and clicks save, the app keeps the task unassigned in the later column, but it should require an owner before the task can leave To do.

### 3. Duplicate titles
**Severity:** Medium

When a user adds a task with a title that already exists on the board, the app creates a second card with the same name, but it should reject duplicate titles in that project.

### 4. Description not required
**Severity:** Medium

When a user adds a task with only a title, the app creates the task with no description field, but it should require a description before the task can be added.

### 5. Status dropdown labels
**Severity:** Medium

When a user opens the add-a-task status dropdown, the app shows `todo`, `in_progress`, `review`, and `done`, but it should show the same labels as the board columns (To do, In progress, In review, Done).

### 6. Viewer still sees edit actions
**Severity:** Low

When a viewer opens a project or a task, the app still shows Add a task, Save, and Delete task, but it should hide those controls so a viewer can only read the board.

# Non Functional

### 1. Session lives in `localStorage` for 30 days
**File:** `src/lib/api-client.ts` (lines 3–27); `src/lib/jwt.ts` (lines 7, 14–16)
**Category:** Security
**Severity:** High

When a user signs in, the app stores the JWT in `localStorage` (`taskboard_token`) with a 30-day expiry and no httpOnly cookie or refresh rotation. Any XSS on the origin can steal that token and act as the user until it expires. Sessions should be short-lived, cookie-based, and not readable by page script.


### 2. User email is not unique; login uses the first match
**File:** `prisma/schema.prisma` (lines 23–26); `src/app/api/auth/login/route.ts` (line 16); `src/app/api/auth/register/route.ts` (lines 17–24)
**Category:** Data Integrity
**Severity:** High

When two `users` rows share an email (allowed because there is no `@@unique` on `email`), register can race past `findFirst`, and login authenticates whichever row Prisma returns first. Account identity, password resets, and “who did this” on tasks then point at the wrong person. Email must be unique at the database, and login should use `findUnique`.



