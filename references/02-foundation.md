# Foundation phase: Layers 3–5

## Contents
- Layer 3: Databases & storage
- Layer 4: Auth & permissions
- Layer 5: APIs & backend logic

---

## Layer 3: Databases & storage

### What it is
Where data lives: a relational database for structured data, object storage (S3, R2, Supabase Storage) for files, sometimes a cache or queue.

### What agents get wrong
- No migrations; edit schemas directly or regenerate from scratch.
- Missing indexes: fine at 100 rows, slow at 100,000.
- No `tenant_id` / `organization_id` on tables, causing data leaks between customers.
- Storing files in the database, or making uploads publicly readable.
- No backups, no soft deletes, no `created_at` / `updated_at`.

### Decisions the human owns
- Database: Postgres is the safe default for almost any SaaS.
- Hosted provider (Supabase, Neon, RDS, PlanetScale, etc.).
- Backup and retention policy.
- What gets deleted when a user deletes their account.

### Prompts

**3a. Schema design**
```
Based on /docs/system-design.md, design the database schema.
Requirements:
- Multi-tenant: every tenant-owned table has organization_id, with a foreign key
- Every table has id (UUID), created_at, updated_at
- Use soft deletes (deleted_at) where users might want to recover data
- Add indexes for every foreign key and every column we filter or sort by
- Use proper constraints (NOT NULL, UNIQUE, CHECK) instead of relying on app code
- Money stored as integer minor units (paise/cents), never floats
Output: an ERD as a Mermaid diagram, then the schema in our ORM format,
then an explanation of each index. Don't run migrations yet.
```

**3b. Migrations and seed data**
```
Create the initial migration from the approved schema. Then write a seed
script that creates 2 organisations, each with 3 users and realistic sample
data, so I can test that tenants can't see each other's data.
Run the migration and seed against the local database and show me the output.
```

**3c. File storage**
```
Implement file uploads using [S3 / R2 / Supabase Storage].
- Files are private by default
- Upload via pre-signed URLs directly from the browser (don't stream through our server)
- Validate file type and size on the server before issuing the URL
- Store only the file key and metadata in the database
- Downloads use short-lived signed URLs, and only after an authorization check
- Scope storage paths by organization_id
Write tests for: wrong file type, too large, and user from another org trying to download.
```

**3d. Performance review**
```
Review every database query in the codebase. Flag:
- N+1 queries
- queries without a supporting index
- SELECT * where we only need a few columns
- missing pagination on list endpoints
- any query that doesn't filter by organization_id on a tenant table
Give me a table: file, line, problem, fix. Then fix the high-severity ones.
```

**3e. Backups**
```
Document our backup setup in /docs/runbooks/backups.md: how often backups run,
how long they're kept, and step-by-step how to restore to a point in time.
Then write a script that restores the latest backup into a separate
database so we can test restores without touching production.
```

### Verify
- [ ] Every schema change is a migration file in version control.
- [ ] Logged in as Org A, change an ID in the URL to Org B's resource. It must fail.
- [ ] A backup has actually been restored at least once.

---

## Layer 4: Auth & permissions

### What it is
**Authentication** = who are you (login, signup, sessions, password reset, SSO). **Authorization** = what are you allowed to do (roles, permissions, which organisation's data you can see).

### What agents get wrong
- Hand-rolling password hashing and session logic. Use a proven provider or library.
- Checking permissions only in the frontend (hiding a button) instead of on the server.
- Checking that a user is logged in but not that they own the resource requested. This is the single most common SaaS vulnerability (IDOR / broken object-level authorization).
- Forgetting email verification, password-reset expiry and session revocation.

### Decisions the human owns
- Auth provider: Clerk, Auth0, Supabase Auth, Auth.js, WorkOS (if enterprise SSO is needed).
- Roles: start simple, e.g. Owner, Admin, Member, Viewer.
- Login methods: email + password, magic link, Google, SSO.

### Prompts

**4a. Authentication**
```
Implement authentication using [provider]. Do not hand-roll password hashing
or session management.
Include: signup, login, logout, email verification, password reset,
Google login, and session expiry. Protect all routes under /app and /api
except the ones I list: [public routes].
Add a middleware that attaches the current user and current organization
to every request.
```

**4b. Authorization model**
```
Design a role-based permission system for these roles: [Owner, Admin, Member, Viewer].
1. Write a permissions matrix (role × action) as a table in /docs/permissions.md.
2. Implement a single central function, e.g. can(user, action, resource),
   that every endpoint uses. No ad-hoc role checks scattered around.
3. Every read or write of a tenant resource must verify the resource's
   organization_id matches the user's current organization.
4. Frontend hides actions the user can't take, but the server is the
   source of truth.
```

**4c. Authorization tests (critical)**
```
Write integration tests that prove, for every API endpoint:
- unauthenticated requests get 401
- authenticated users without permission get 403
- a user from Org A gets 404 (not 403) when requesting Org B's resource by ID
- each role can do exactly what /docs/permissions.md says, nothing more
Generate these tests from the permissions matrix so they stay in sync.
```
(404 rather than 403 for cross-tenant requests avoids confirming that the resource exists.)

**4d. Team features**
```
Implement: invite a teammate by email, accept invite, change role,
remove member, transfer ownership, and leave organization.
Edge cases to handle: last owner can't leave, invite links expire after
7 days, removed members lose access immediately (revoke their sessions).
```

### Verify
- [ ] In DevTools, copy an API request and change the resource ID to another org's. Response is 404.
- [ ] A Viewer calling a delete endpoint directly (not via UI) gets 403.
- [ ] No file contains custom password hashing code.

---

## Layer 5: APIs & backend logic

### What it is
The server-side rules of the product: endpoints, validation, business logic, background jobs, webhooks, and integrations like payments.

### What agents get wrong
- No input validation; trusting whatever the client sends.
- Inconsistent error formats across endpoints.
- Slow work (emails, third-party calls) inside the request.
- Non-idempotent payment and webhook handling → double charges, duplicate records.
- No pagination on list endpoints.

### Decisions the human owns
- REST vs tRPC vs GraphQL (REST or tRPC is plenty for most).
- Payment provider: Stripe, Razorpay (India), Paddle / Lemon Squeezy (merchant of record handles tax).
- Background job system: Inngest, Trigger.dev, BullMQ, Cloud Tasks, etc.

### Prompts

**5a. API conventions**
```
Define our API conventions and write them to /docs/api-conventions.md, then
add a summary to CLAUDE.md. Include:
- URL and naming style
- a single error response format with error codes
- validation with [Zod / Pydantic] on every input, at the boundary
- pagination format (cursor-based) for all list endpoints
- how auth and org context are read in every handler
- HTTP status codes we use and when
Then refactor existing endpoints to match.
```

**5b. Build a feature endpoint**
```
Build the API for [feature, e.g. "projects": create, list, get, update, delete].
Follow /docs/api-conventions.md and /docs/permissions.md.
- Business logic goes in /services, route handlers stay thin
- Validate every input
- Authorization check on every operation via can()
- Filter every query by organization_id
- Cursor pagination on list
Write unit tests for the service and integration tests for each endpoint,
including permission and cross-tenant cases.
```

**5c. Background jobs**
```
Set up background jobs using [tool]. Move these out of the request path:
[sending emails, generating exports, calling third-party APIs].
Each job must: be idempotent (safe to run twice), retry with exponential
backoff, have a max retry count, and log failures with enough context
to debug. Add a way to see failed jobs.
```

**5d. Payments and webhooks**
```
Integrate [Stripe / Razorpay] subscriptions for these plans: [plans].
- Treat webhooks as the source of truth for subscription state, not the
  redirect after checkout
- Verify webhook signatures
- Make webhook handlers idempotent by storing processed event IDs
- Handle: payment success, failure, cancellation, plan change, refund
- Gate features by plan with a single helper, e.g. hasFeature(org, feature)
Write tests that replay each webhook event twice and assert no duplicates.
Use test mode keys only.
```

### Verify
- [ ] Send garbage JSON to an endpoint. Clean 400 in the standard error format, no stack trace.
- [ ] Replay the same webhook twice. Nothing duplicates.
- [ ] Any request taking more than ~1 second has a reason or has moved to a background job.
