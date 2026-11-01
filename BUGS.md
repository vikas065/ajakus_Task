Top 4 Findings:



# Top 4 Findings

1. When an authenticated user can update any task in the system without checking project membership or role permissions, the app exposes an authorization bypass, but it should enforce project-scoped role checks before every task mutation.
   - File/Line: src/app/api/tasks/[id]/route.ts:10-29
   - Category: Security
   - Severity: Critical
   - Description: The PATCH endpoint updates tasks by ID without validating whether the caller belongs to the same project or has a permitted role. This creates an access-control weakness that allows unauthorized task edits and can compromise business workflow integrity.

2. When user search input is passed directly into a raw SQL query, the app allows SQL injection, but it should use parameterized queries and validated input sanitization for all dynamic database access.
   - File/Line: src/app/api/projects/[id]/tasks/route.ts:10-26
   - Category: Security
   - Severity: High
   - Description: The q parameter is concatenated into a Prisma $queryRawUnsafe statement, which is vulnerable to SQL injection. A malicious user could query, manipulate, or expose project data beyond intended scope.

3. When the app stores JWTs and user data in browser localStorage, the app leaves sensitive session data exposed to client-side scripts, but it should keep tokens in secure, short-lived server-side session mechanisms or hardened storage.
   - File/Line: src/lib/api-client.ts:3-21
   - Category: Security
   - Severity: High
   - Description: Storing the token and user object in localStorage makes session hijacking easier if the browser is compromised by XSS or malicious scripts. This is a significant risk because task and project data are sensitive operational information.

   
4. When a user tries to create a task with a title that already exists, the app allows duplicate task titles, but it should prevent duplicate titles and show a proper validation message.

# UI Findings

1. When a user tries to create a task with a title that already exists, the app allows duplicate task titles, but it should prevent duplicate titles and show a proper validation message.
2. When a user moves a task to another stage without adding a description, the app allows the action, but it should require a description before the task can proceed beyond Todo.
3. When a task is moved to a different stage, the app leaves it in the Unassigned state, but it should automatically assign it to a valid assignee or clear the Unassigned state based on the workflow rules.
4. When a user tries to create a task from a stage other than Todo, the app allows the task to be created, but it should only allow task creation from the Todo stage.
5. When a task is selected and the dropdown name is compared with the task header, the app shows inconsistent labels, but it should align the dropdown name with the task header name.
6. When a Viewer user tries to assign a task, the app allows the action, but it should restrict assignment access to authorized roles only.
7. When a Viewer user tries to move a task to another stage, the app allows the action, but it should restrict stage movement permissions to authorized roles only.
8. When an admin email and password are stored in the application, the app exposes sensitive credentials, but it should never store or display admin credentials in the application and should use secure authentication methods.
9. When a task has reached the Done stage, the Delete option is still visible, but it should not be available after the task is completed.

# Non Functional Findings

1. When an authenticated user can update any task in the system without checking project membership or role permissions, the app exposes an authorization bypass, but it should enforce project-scoped role checks before every task mutation.
   - File/Line: src/app/api/tasks/[id]/route.ts:10-29
   - Category: Security
   - Severity: Critical
   - Description: The PATCH endpoint updates tasks by ID without validating whether the caller belongs to the same project or has a permitted role. This creates an access-control weakness that allows unauthorized task edits and can compromise business workflow integrity.

2. When the app stores JWTs and user data in browser localStorage, the app leaves sensitive session data exposed to client-side scripts, but it should keep tokens in secure, short-lived server-side session mechanisms or hardened storage.
   - File/Line: src/lib/api-client.ts:3-21
   - Category: Security
   - Severity: High
   - Description: Storing the token and user object in localStorage makes session hijacking easier if the browser is compromised by XSS or malicious scripts. This is a significant risk because task and project data are sensitive operational information.

3. When user search input is passed directly into a raw SQL query, the app allows SQL injection, but it should use parameterized queries and validated input sanitization for all dynamic database access.
   - File/Line: src/app/api/projects/[id]/tasks/route.ts:10-26
   - Category: Security
   - Severity: High
   - Description: The q parameter is concatenated into a Prisma $queryRawUnsafe statement, which is vulnerable to SQL injection. A malicious user could query, manipulate, or expose project data beyond intended scope.

4. When duplicate task titles are allowed without uniqueness rules, the app creates ambiguous work items and weakens data integrity, but it should enforce valid business rules for task naming and required task metadata.
   - File/Line: prisma/schema.prisma:54-75 and src/schemas/task.ts:5-21
   - Category: Data Integrity
   - Severity: Medium
   - Description: The schema does not enforce uniqueness on task titles per project, and the description field is optional. This makes task reporting and follow-up unreliable and can lead to duplicate or incomplete records in active workflow stages.

5. When the project detail endpoint fetches all memberships and tasks in one large response, the app scales poorly as data grows, but it should paginate or filter nested collections to protect performance and responsiveness.
   - File/Line: src/app/api/projects/[id]/route.ts:18-33
   - Category: Performance
   - Severity: Medium
   - Description: The route loads all nested task and membership data in a single operation, which increases payload size and database cost. As project volume grows, this will slow dashboard performance and degrade user experience.

6. When authorization and task lifecycle rules are spread across route handlers without shared domain logic, the app becomes harder to secure and maintain, but it should centralize RBAC and workflow validation in reusable service-layer checks.
   - File/Line: src/app/api/projects/[id]/route.ts:18-33 and src/app/api/projects/[id]/tasks/route.ts:10-39
   - Category: Architecture
   - Severity: Medium
   - Description: Business rules for membership, task creation, and project editing are fragmented across endpoints. This increases maintenance risk and makes it easier for inconsistent authorization logic to be introduced across screens and APIs.