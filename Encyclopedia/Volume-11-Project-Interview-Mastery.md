# THE COMPLETE .NET DEVELOPER ENCYCLOPEDIA + INTERVIEW MASTERY GUIDE
# Volume 11: Project Interview Mastery

> **Target Audience:** .NET Backend Developer (~3 years exp) | Infosys Interview Preparation  
> **Written by:** Senior Technical Interviewer & .NET Principal Engineer  
> **Estimated Read Time:** 3–4 hours | **Practice Time Required:** 10–15 hours

---

## TABLE OF CONTENTS

1. [How Interviewers Think](#1-how-interviewers-think)
2. [The Project Description Framework (BARCC)](#2-the-project-description-framework-barcc)
3. [Architecture Walkthrough Questions](#3-architecture-walkthrough-questions)
4. [Authentication Implementation Scripts](#4-authentication-implementation-scripts)
5. [Error Handling & Exception Management](#5-error-handling--exception-management)
6. [Database Migrations in Production](#6-database-migrations-in-production)
7. [Performance Issues & Diagnosis](#7-performance-issues--diagnosis)
8. [API Security Deep Dive](#8-api-security-deep-dive)
9. [Concurrency & Race Conditions](#9-concurrency--race-conditions)
10. [Deployment Process & DevOps](#10-deployment-process--devops)
11. [Debugging Production Issues](#11-debugging-production-issues)
12. [Third-Party API Failure Handling](#12-third-party-api-failure-handling)
13. [Behavioral + Technical Hybrid Questions](#13-behavioral--technical-hybrid-questions)
14. [The Infosys Interview Strategy](#14-the-infosys-interview-strategy)
15. [Resume Line-by-Line Preparation](#15-resume-line-by-line-preparation)
16. [Common Candidate Mistakes](#16-common-candidate-mistakes)

---

## 1. HOW INTERVIEWERS THINK

### The Interviewer's Mental Model

When a technical interviewer asks you a question, they are not simply checking if you know the answer. They are running a multi-dimensional evaluation in their head:

```
Can this person work on our codebase?
├── Do they understand the WHY, not just the WHAT?
├── Can they reason under pressure?
├── Will they cause problems or solve them?
└── Are they aware of trade-offs?
```

### What Depth vs. Breadth Means in Practice

**Breadth candidate (weak signal):** "Yes, I've used JWT. I set up authentication in my project."

**Depth candidate (strong signal):** "We used JWT with RS256 signing because we had multiple services that needed to validate tokens without sharing a secret. The issuing service holds the private key; validators only need the public key. We store refresh tokens in the database with expiry and a revocation flag so we can invalidate sessions. One thing we had to handle carefully was token expiry race conditions — if a client sends two requests simultaneously when the token is about to expire, both might try to refresh, so we used a distributed lock via Redis."

> **Key insight:** The depth candidate didn't just say what they did — they explained the problem they were solving, the alternatives they considered, and the edge cases they handled.

### The Five Signals Interviewers Evaluate

| Signal | What They Look For | Red Flag |
|--------|-------------------|----------|
| **Technical Accuracy** | Is the answer factually correct? | Wrong facts stated with confidence |
| **Depth of Understanding** | Do they know WHY it works? | "It just works" answers |
| **Problem-Solving Approach** | Can they reason through a new problem? | Memorized answers only |
| **Communication Clarity** | Can they explain complex things simply? | Jargon without explanation |
| **Self-Awareness** | Do they know what they don't know? | Bluffing on unknown topics |

### The Interviewer's Trap Technique

Experienced interviewers use a technique called **"Peel the Onion"**:
1. Ask a standard question
2. When you answer, go one layer deeper
3. Keep going until you reach your knowledge boundary
4. Observe how you behave at the boundary

**The right behavior at your knowledge boundary:**  
"That's at the edge of my knowledge. I know that [what you DO know], and from there I would reason that [logical extension], but I haven't validated this in production."

**Wrong behavior:** Making something up or going silent.

### What Infosys Specifically Looks For

Infosys interviewers are known to:
- Read your resume **line by line** — every technology you mention is fair game
- Ask you to **write code** on screen or a shared editor (no IDE assistance)
- Ask **"what's wrong with this code"** pattern questions
- Focus heavily on: OOP, C#, ASP.NET Core, EF Core, SQL, Design Patterns
- Test DI service lifetimes with trick questions
- Expect you to explain async/await internals, not just "it makes code async"

---

## 2. THE PROJECT DESCRIPTION FRAMEWORK (BARCC)

### Why You Need a Framework

Most candidates describe their projects chronologically ("First I did X, then I did Y..."). This wastes the interviewer's time and buries your most impressive work. Use the **BARCC framework** instead.

### BARCC Framework

```
B - Business Context    (What problem does the system solve? For whom?)
A - Architecture        (High-level system design: layers, services, data stores)
R - Role & Ownership    (What specifically DID YOU own and build?)
C - Key Challenges      (What was hard? What did you solve?)
C - Critical Decisions  (Why X instead of Y? What trade-offs did you accept?)
```

### The 90-Second Project Pitch (Memorize This Structure)

**Template:**
```
"[Project Name] is a [type of system] that [core purpose] for [user/business].

The architecture is [monolith/microservices/hybrid] with [key layers/services].
We use [primary database] for [purpose] and [secondary store] for [purpose].

My specific ownership was [concrete things you built/owned].

The most interesting technical challenge I solved was [specific problem], 
which I addressed by [specific solution], which resulted in [measurable outcome].

One key decision we made was [decision], because [reason], and the trade-off 
we accepted was [trade-off]."
```

### Sample Answers for Common Project Types

#### Sample A: E-Commerce / Order Management System

> "I worked on an order management system for a retail client handling roughly 50,000 orders per day. The architecture is a layered monolith — Presentation (ASP.NET Core Web API), Application (CQRS with MediatR), Domain (DDD entities), Infrastructure (EF Core + SQL Server). We also have Redis for session caching and a separate Reporting service.
>
> My ownership was the order processing pipeline — the core business logic from order placement through payment validation, inventory check, and fulfillment dispatch. I also built the integration with the third-party payment gateway.
>
> The hardest challenge was handling concurrent order updates. Two warehouse workers could update the same order simultaneously, causing inconsistent state. I implemented optimistic concurrency using EF Core's RowVersion concurrency token, with a retry policy using Polly for transient conflicts.
>
> A key architectural decision was using CQRS even though it's a monolith. We separated reads and writes so the reporting queries could use denormalized read models without impacting the command path. The trade-off was more code — separate command handlers and query handlers — but it paid off when we needed to optimize read performance independently."

#### Sample B: HR / Employee Management System

> "I built a module within an HR platform that manages employee onboarding, leave management, and payroll integration. The system is an ASP.NET Core Web API following Clean Architecture — Domain, Application, Infrastructure, and API layers. SQL Server is the primary store, and we have a file store on Azure Blob for documents.
>
> I owned the leave management module end-to-end — leave application, approval workflows, balance calculation, and the reporting dashboard API.
>
> The most interesting problem was the leave balance calculation. Leave accrues monthly with carry-forward rules that vary by leave type and employee grade. I modeled this as a domain service with configurable policies, which allowed HR to change rules without code deployments.
>
> We chose Repository pattern over direct DbContext use because we had a requirement to switch databases in a future phase. In practice, that never happened, so in hindsight a simpler approach with direct EF Core queries would have been fine. That's a lesson I carry — don't over-engineer for future requirements that aren't confirmed."

#### Sample C: API / Integration Platform

> "I worked on an API integration layer that acts as a facade between our internal microservices and external client applications. It's essentially a custom API gateway — request routing, authentication, rate limiting, and response transformation.
>
> The architecture is an ASP.NET Core middleware pipeline — each request goes through: JWT validation → rate limit check → routing → service call → response mapping. We use Redis for rate limiting counters and token blacklisting.
>
> I owned the authentication/authorization module and the rate limiting system. The rate limiting was particularly interesting — we needed per-client rate limits with sliding window semantics. I implemented it using the Lua scripting capability of Redis to make the check-and-increment atomic, avoiding race conditions.
>
> The critical decision was using Redis Lua scripts rather than a distributed lock with .NET code. The lock approach would have added latency; the Lua script runs atomically on the Redis server in a single round-trip. The trade-off is that Redis Lua is opaque — debugging it is harder — but correctness justified it."

---

## 3. ARCHITECTURE WALKTHROUGH QUESTIONS

### Q: "Walk me through your application architecture."

This is the most common opening question. The interviewer wants to see if you have a mental model of your system — not just the technology list.

#### How to Answer: Layered Monolith

**Structure your answer in layers:**

```
"Our application follows a [N-layer / Clean Architecture / Onion] pattern.

[API Layer]: ASP.NET Core Web API — handles HTTP, routing, auth, validation.
[Application Layer]: Business orchestration — MediatR handlers, DTOs, service interfaces.
[Domain Layer]: Core business entities, domain services, value objects.
[Infrastructure Layer]: EF Core, SQL Server, third-party integrations.

Data flows top-down: A request hits the API layer, is validated by FluentValidation, 
passed to an Application layer handler via MediatR, which calls domain logic and 
persists via the Infrastructure layer.

Dependency rule: Inner layers know nothing about outer layers. 
The Domain doesn't reference EF Core. The Application doesn't reference the API.
This means we can test domain logic without any database."
```

**What NOT to say:**
- ❌ "I used .NET and SQL Server" (no architecture)
- ❌ Listing technologies without explaining how they relate
- ❌ "We have controllers and services" (too vague)

#### How to Answer: Microservices

```
"We have [N] microservices, each owning its data store. Services communicate 
via REST for synchronous calls and RabbitMQ for asynchronous events.

Key services: [User Service], [Order Service], [Notification Service].
Each service is independently deployable with its own CI/CD pipeline.

We use an API Gateway (Ocelot) as the entry point — it handles auth, routing,
and rate limiting. Internal service-to-service calls use named HttpClient with 
Polly retry and circuit breaker policies.

The hardest architectural challenge was distributed transactions — when Order Service
and Inventory Service both need to update atomically. We solved this with the Saga 
pattern using choreography — each service publishes events and subscribes to 
compensating events if something fails downstream."
```

### Q: "Why did you choose [technology/pattern]?"

Use the **Problem → Options → Decision → Trade-offs** (PODT) framework:

```
Problem:    "We had [specific problem]"
Options:    "We considered [option A] and [option B]"
Decision:   "We chose [option] because [specific reasons]"
Trade-offs: "We accepted the trade-off of [downside] because [justification]"
```

#### Why CQRS?

> "Problem: Our read patterns (reporting, dashboards) were slowing down our write path because they shared the same complex domain model. Every report query was loading entire aggregates just to display a few fields.  
> Options: We considered query optimization with raw SQL views, or full CQRS with separate read models.  
> Decision: We chose CQRS because the performance issue was architectural — the read and write models fundamentally had different shapes. Read models can be flat and denormalized; write models need to enforce invariants.  
> Trade-off: More code. Every feature now has command handlers and query handlers. Junior developers found it harder to follow. We mitigated this with good folder structure and conventions."

#### Why Repository Pattern?

> "Problem: We wanted to abstract data access so our Application layer didn't depend on EF Core directly.  
> Options: Direct DbContext, Repository, Unit of Work.  
> Decision: Repository because it gives us a clean seam for unit testing — we can mock IOrderRepository without spinning up a real database.  
> Trade-off: We lose EF Core's rich querying capability at the application layer. Complex queries that would be one LINQ expression become custom methods on the repository. We accepted this because testability was a priority."

#### Why EF Core instead of Dapper?

> "Problem: We needed a data access layer that could handle complex domain model mapping and enforce business rules at the persistence layer.  
> Options: EF Core (full ORM), Dapper (micro-ORM + raw SQL), or ADO.NET.  
> Decision: EF Core for its Code-First migrations, change tracking, and the ability to map complex relationships. Dapper would require hand-writing SQL for every operation.  
> Trade-off: EF Core generates suboptimal SQL for complex queries. We added Dapper for reporting queries where we needed precise SQL control. So the final answer is: EF Core for writes and simple reads, Dapper for complex reporting queries."

#### Why MediatR?

> "Problem: Our controllers were becoming fat — they were orchestrating business logic directly and had multiple dependencies.  
> Options: Service layer pattern, MediatR CQRS, or thin controllers with direct domain calls.  
> Decision: MediatR because it decouples controllers from business logic. Each command/query is a standalone class — easy to test, easy to find, and easy to add cross-cutting concerns via pipeline behaviors (logging, validation, transactions).  
> Trade-off: It adds indirection. Tracing from a controller action to the actual handler requires knowing the MediatR pattern. New team members needed onboarding."

---

## 4. AUTHENTICATION IMPLEMENTATION SCRIPTS

### Complete JWT Flow Explanation Script

**Interviewer:** "Explain how you implemented authentication in your project."

**Your Answer:**

> "We use JWT Bearer authentication with an RS256 signing algorithm.
>
> **The flow:**
> 1. User submits credentials (username/password) to the `/auth/login` endpoint.
> 2. We validate credentials against the database — passwords are stored as bcrypt hashes.
> 3. On success, we generate a JWT: payload contains `userId`, `roles`, `email`, and standard claims (`iss`, `aud`, `exp`, `iat`). We sign it with our RSA private key.
> 4. We also generate a refresh token — a cryptographically random 64-byte value stored in the database with `userId`, `expiry`, `createdAt`, and an `isRevoked` flag.
> 5. Both tokens are returned to the client. The access token is short-lived (15 minutes). The refresh token is long-lived (7 days).
>
> **On subsequent requests:**  
> The client sends the JWT in the `Authorization: Bearer <token>` header. ASP.NET Core's JWT middleware validates the signature using our public key, checks expiry, and populates `HttpContext.User`. Our authorization policies then check roles and claims.
>
> **Token refresh:**  
> When the access token expires, the client sends the refresh token to `/auth/refresh`. We look up the refresh token in the database, check it's not expired or revoked, generate a new access token, and optionally rotate the refresh token (issue a new one and revoke the old).
>
> **Why RS256 instead of HS256?**  
> With HS256, every service that validates a token needs the shared secret — if any service is compromised, the secret is exposed. RS256 uses asymmetric keys — only the auth service needs the private key; validators only need the public key. This is safer in a multi-service environment."

### "How do you invalidate a JWT?" — The Follow-Up Trap

This is a deliberate trap. JWTs are stateless — you can't "recall" them. Interviewers love this question.

**Wrong answer:** "I delete it from the database." (JWTs aren't stored in the database.)

**Right answer:**

> "JWT invalidation is fundamentally at odds with the stateless nature of JWTs. A valid JWT will be accepted by any validator until it expires, regardless of what's in your database.
>
> There are three practical approaches:
>
> **1. Short Expiry (Preferred):** Keep access tokens short-lived (15 minutes). If you need to "logout" a user, the token is invalid in at most 15 minutes anyway. Invalidate the refresh token in the database so they can't get a new access token.
>
> **2. Token Blacklist / Jti Claim:** Every JWT includes a `jti` (JWT ID) — a unique identifier. On logout, you store the `jti` in a Redis blacklist with a TTL matching the token's remaining lifetime. Every request checks if the `jti` is blacklisted. Downside: this re-introduces statefulness.
>
> **3. Token Versioning:** Store a `tokenVersion` integer per user in the database. Embed this version in the JWT claim. On every request, validate the JWT and also check that the version matches the database. On logout or password change, increment the version. All existing tokens become invalid. Downside: one database read per request.
>
> In our project, we used approach 1 (short expiry) with refresh token revocation, because the use case didn't require instant session revocation."

---

## 5. ERROR HANDLING & EXCEPTION MANAGEMENT

### Complete Script: "How do you handle errors and exceptions in your API?"

> "We have three layers of error handling:
>
> **1. Global Exception Middleware:**  
> We have a custom middleware class that wraps the entire pipeline in a try-catch. When an unhandled exception reaches this middleware, it:
> - Logs the full exception with structured logging (Serilog) — including correlation ID, user ID, and request path.
> - Returns a `ProblemDetails` JSON response (RFC 7807 standard) with a status code and a user-safe message. The stack trace NEVER goes to the client.
>
> ```csharp
> public async Task InvokeAsync(HttpContext context)
> {
>     try { await _next(context); }
>     catch (Exception ex)
>     {
>         _logger.LogError(ex, "Unhandled exception for {Path}", context.Request.Path);
>         context.Response.StatusCode = 500;
>         await context.Response.WriteAsJsonAsync(new ProblemDetails
>         {
>             Status = 500,
>             Title = "An unexpected error occurred.",
>             Instance = context.TraceIdentifier
>         });
>     }
> }
> ```
>
> **2. Domain Exception Mapping:**  
> We have custom exception types: `NotFoundException`, `ValidationException`, `ConflictException`. The middleware maps these to appropriate HTTP status codes (404, 422, 409) before they reach the default catch. This keeps business logic clean — domain code throws meaningful exceptions without knowing about HTTP.
>
> **3. Validation at Entry:**  
> We use FluentValidation with ASP.NET Core automatic validation. Invalid requests are rejected with 422 and a structured error response before they reach business logic.
>
> **Why ProblemDetails?**  
> It's the RFC 7807 standard — consistent, parseable, and clients know what to expect. It includes `type`, `title`, `status`, `detail`, and `instance` (correlation ID for tracing)."

### Logging Strategy

> "We use Serilog with structured logging. Every log entry includes:
> - `CorrelationId` — a GUID we inject at the middleware level and pass through via `AsyncLocal` or `ILogger` scope
> - `UserId` — from the JWT claim if authenticated
> - `RequestPath`, `StatusCode`, `ElapsedMs` — for every request (logged by our request logging middleware)
> - Exceptions are logged at `Error` level with full stack trace
>
> Sinks: In development, we log to console. In production, we log to Application Insights (for structured querying) and optionally to Elasticsearch via Kibana for long-term storage.
>
> **Critical rule we follow:** Never log sensitive data — no passwords, no PII in logs, no credit card numbers. We have a custom destructuring policy in Serilog that scrubs known sensitive properties."

---

## 6. DATABASE MIGRATIONS IN PRODUCTION

### Complete Script: "How do you handle database migrations in production?"

> "Database migrations in production require more care than in development. Here's our approach:
>
> **Development workflow:**
> We use EF Core Code-First migrations. Every schema change is captured in a migration file — a C# file with `Up()` and `Down()` methods. We commit migration files to source control alongside the code change.
>
> **Production migration process:**
>
> **Step 1: Migration review.**  
> Before deployment, we review the generated SQL with `dotnet ef migrations script` to generate a pure SQL script. We check for table locks (ALTER TABLE on large tables), data-loss operations (DROP COLUMN), and long-running operations.
>
> **Step 2: Backward-compatible migrations only.**  
> We never do a breaking migration and code deployment simultaneously. The process is: deploy migration → deploy code. Or in reverse: deploy new code that handles both old and new schema → deploy migration → remove backward-compatibility code.
>
> Example: Renaming a column is dangerous. Instead: add the new column → deploy code that writes to both columns and reads from new → drop old column in a later migration.
>
> **Step 3: Apply migration separately.**  
> We use `MigrateAsync()` in a separate pre-deployment step (a job or script), NOT in `Program.cs` startup. Calling `db.Database.MigrateAsync()` in startup means every instance tries to run migrations simultaneously on startup — only one should run them.
>
> **Step 4: Health check after migration.**  
> We have a health check endpoint that validates the database schema version matches the expected version. Deployment pipelines check this before routing traffic.
>
> **If a migration fails:**  
> EF Core migrations run in a transaction where the database supports it (SQL Server does). If the migration fails mid-way, it rolls back. The `__EFMigrationsHistory` table only gets an entry on success. We can re-run the deployment after fixing the issue."

### Safe Migration Cheat Sheet

| Operation | Risk | Safe Approach |
|-----------|------|--------------|
| Add nullable column | Low | Direct migration |
| Add non-nullable column with default | Medium | Add with default → backfill → add constraint |
| Rename column | High | Add new → copy data → update code → drop old |
| Drop column | High | Deprecate in code first → drop in later release |
| Add index | Medium | Use `ONLINE = ON` in SQL Server to avoid table lock |
| Change column type | High | New column → migrate data → update code → drop old |

---

## 7. PERFORMANCE ISSUES & DIAGNOSIS

### Complete Script: "How do you handle performance issues?"

> "When we notice a performance issue, I follow a structured diagnosis process rather than guessing:
>
> **Step 1: Measure first, optimize second.**  
> We never optimize without data. We look at metrics first: response time percentiles (p95, p99), error rates, database query duration, memory usage. Application Insights gives us transaction traces so we can see exactly which part of the request is slow.
>
> **Step 2: Identify the bottleneck.**  
> Is it CPU? Memory? Database? External API? Network? Usually it's one of:
> - **N+1 queries** — loading related data in a loop instead of a single JOIN
> - **Missing database index** — full table scans on large tables
> - **No caching** — expensive computations or queries repeated unnecessarily
> - **Synchronous blocking** — thread pool starvation from blocking async code
>
> **Step 3: Fix by category:**
>
> **Database issues:**
> - Use `Include()` to fix N+1 problems
> - Add indexes for frequent query predicates
> - Use `AsNoTracking()` for read-only queries
> - Use `Select()` to project only needed columns — avoid loading entire entities
> - For complex reports, use raw SQL or Dapper
>
> **Caching:**
> - In-memory cache (`IMemoryCache`) for single-instance data with short TTL
> - Distributed cache (Redis via `IDistributedCache`) for data shared across instances
> - Response caching for public, rarely-changing API responses
> - Cache-aside pattern: check cache → if miss, load from DB → store in cache
>
> **Async:**
> - Never call `.Result` or `.Wait()` on Tasks — it blocks thread pool threads
> - Use `ConfigureAwait(false)` in library code
> - Ensure all I/O operations are async (DB, file, HTTP)
>
> **Step 4: Validate the fix.**  
> Run a load test (k6 or JMeter) against the change. Compare p95 response times before and after. Don't ship a performance fix without proof it worked."

### "What is N+1? How do you fix it?"

> "N+1 is when you execute 1 query to fetch N records, then execute N additional queries to fetch related data — one per record. With 1000 orders, that's 1001 database round-trips instead of 1 or 2.
>
> **Example in EF Core:**
> ```csharp
> // N+1 problem
> var orders = await _context.Orders.ToListAsync(); // 1 query
> foreach (var order in orders)
> {
>     var items = order.OrderItems; // N queries (lazy loading)
> }
>
> // Fixed with eager loading
> var orders = await _context.Orders
>     .Include(o => o.OrderItems)
>     .ToListAsync(); // 1 query with JOIN
> ```
>
> EF Core can generate the N+1 pattern through lazy loading. We disable lazy loading in production and use explicit `Include()` or split queries where appropriate."

---

## 8. API SECURITY DEEP DIVE

### Complete Script: "How do you ensure API security?"

> "API security is layered — there's no single control. We implement defense in depth:
>
> **1. Authentication & Authorization:**  
> JWT authentication validates every request. Role-based and policy-based authorization controls what authenticated users can do. We follow least-privilege — users get only what they need.
>
> **2. Input Validation:**  
> Every incoming DTO is validated with FluentValidation. We validate data types, ranges, length limits, and format (regex for emails, etc.). We never trust client input — SQL injection is impossible with parameterized queries (EF Core does this by default), but we still validate to protect business logic.
>
> **3. Rate Limiting:**  
> We use ASP.NET Core's built-in rate limiting middleware (introduced in .NET 7) to limit requests per client per window. This prevents brute-force attacks on login and DoS attacks on expensive endpoints. For distributed rate limiting across instances, we back the counter with Redis.
>
> **4. CORS:**  
> We configure CORS explicitly — only allowed origins can make cross-origin requests. We never use wildcard (`*`) in production.
>
> **5. HTTPS Everywhere:**  
> All traffic is HTTPS. We use HSTS to instruct browsers to only connect over HTTPS. In development we use the developer cert; in production we use a proper TLS certificate (ACM in AWS or Let's Encrypt).
>
> **6. Security Headers:**  
> We add security headers via middleware: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`. We configure Content Security Policy for any web-facing responses.
>
> **7. Secret Management:**  
> No secrets in code or appsettings.json committed to source control. We use Azure Key Vault (or AWS Secrets Manager) for production secrets, accessed via the `IConfiguration` abstraction. Locally, we use .NET User Secrets.
>
> **8. Sensitive Data Handling:**  
> Passwords are bcrypt-hashed. PII is stored encrypted at rest. Sensitive fields are excluded from logs using Serilog destructuring policies."

---

## 9. CONCURRENCY & RACE CONDITIONS

### Complete Script: "How do you handle concurrency in your application?"

> "Concurrency is a broad topic — we handle it at multiple levels:
>
> **1. Optimistic Concurrency with EF Core:**  
> For entities where concurrent updates are possible but rare, we use EF Core's `[Timestamp]` / `RowVersion` attribute. EF Core adds a `WHERE RowVersion = @originalVersion` to UPDATE statements. If another user has modified the record, the update affects 0 rows, and EF Core throws `DbUpdateConcurrencyException`. We catch this, reload the entity, and either auto-merge or surface a conflict to the user.
>
> ```csharp
> [Timestamp]
> public byte[] RowVersion { get; set; }
> ```
>
> **2. Database Transactions:**  
> When multiple database operations must succeed or fail together, we use explicit transactions:
> ```csharp
> using var transaction = await _context.Database.BeginTransactionAsync();
> try
> {
>     // Multiple operations
>     await _context.SaveChangesAsync();
>     await transaction.CommitAsync();
> }
> catch { await transaction.RollbackAsync(); throw; }
> ```
> We choose the appropriate isolation level: `ReadCommitted` (default, prevents dirty reads), `RepeatableRead` (prevents non-repeatable reads), `Serializable` (prevents phantom reads but has highest contention).
>
> **3. Distributed Locking with Redis:**  
> For distributed scenarios — multiple instances of our service running simultaneously — database transactions aren't enough for non-database operations. We use Redis distributed lock (RedLock algorithm) to ensure only one instance processes a critical section at a time.
>
> Example: Processing a payment — we don't want two instances to process the same payment simultaneously. We acquire a Redis lock on the payment ID before processing.
>
> **4. Async Concurrency — Semaphores:**  
> Within a single process, if we need to limit concurrent access to a resource, we use `SemaphoreSlim`:
> ```csharp
> private readonly SemaphoreSlim _semaphore = new SemaphoreSlim(1, 1);
> await _semaphore.WaitAsync();
> try { /* critical section */ }
> finally { _semaphore.Release(); }
> ```
>
> **5. Idempotency Keys:**  
> For operations that clients might retry (payments, order creation), we require an idempotency key in the request header. We store processed keys in Redis with the operation result. If the same key comes again, we return the cached result without re-processing."

---

## 10. DEPLOYMENT PROCESS & DEVOPS

### Complete Script: "Explain your deployment process."

> "Our deployment pipeline is fully automated using Azure DevOps (or GitHub Actions):
>
> **Pipeline stages:**
> 1. **Source control trigger:** PR to main branch triggers the pipeline
> 2. **Build:** `dotnet build` — compilation errors fail fast
> 3. **Unit tests:** `dotnet test` with coverage threshold. If coverage drops below 80%, the pipeline fails.
> 4. **Integration tests:** Against a Docker-based SQL Server and Redis. Tests are isolated — each test class has its own schema.
> 5. **Code analysis:** SonarQube or Roslyn analyzers for code quality gates
> 6. **Docker build:** Build the Docker image and push to container registry (ACR / ECR)
> 7. **Deploy to Staging:** Helm chart deployment to Kubernetes staging namespace. Run smoke tests and health check validation.
> 8. **Manual approval gate:** Tech lead approves promotion to production
> 9. **Deploy to Production:** Rolling deployment — gradually replace old pods with new ones. If health checks fail, Kubernetes automatically rolls back.
>
> **Environment configuration:**  
> Config is environment-specific and sourced from environment variables (injected by Kubernetes secrets or Azure Key Vault). We use `IConfiguration` abstraction — the app doesn't know whether config comes from appsettings, env vars, or a secret manager.
>
> **Health checks:**  
> Every service exposes `/health/live` (is the process running?) and `/health/ready` (is the service ready to take traffic — DB connected, dependencies up?). Kubernetes uses these as liveness and readiness probes.
>
> **Rolling vs Blue-Green:**  
> Rolling deployment: gradually update pods — safe for backward-compatible changes, zero downtime. Blue-Green: run two identical environments, switch traffic — safer for breaking changes but costs double the infrastructure. We use rolling for most deployments and blue-green for major releases."

---

## 11. DEBUGGING PRODUCTION ISSUES

### Complete Script: "How do you debug production issues?"

> "Production debugging requires structured approach because you can't attach a debugger. Our toolkit:
>
> **Step 1: Structured logs in Application Insights / Kibana.**  
> Every request has a correlation ID. I start by finding the failed request in logs using the correlation ID or error code. Serilog gives us structured properties — we can filter by `UserId`, `OrderId`, `StatusCode`. The full exception with stack trace is in the log.
>
> **Step 2: Distributed Tracing.**  
> For microservices, Application Insights or OpenTelemetry traces show the full request journey across services. I can see which service call was slow or failed — a 504 timeout might originate in Service B, not Service A where the client sees the error.
>
> **Step 3: Metrics and Dashboards.**  
> If the issue is not a specific error but a pattern (response times degrading, error rate increasing), we look at metrics dashboards. CPU, memory, database connection pool exhaustion, thread pool queue depth — these tell the story.
>
> **Step 4: Reproduce locally (safely).**  
> Once I know what triggered the issue, I reproduce it locally with production-like data (sanitized/anonymized). I use feature flags to test with production config without routing real users.
>
> **Step 5: Hotfix process.**  
> Critical bugs get a hotfix branch — minimal change, cherry-picked to main. It goes through the full pipeline but with expedited approvals. We document the incident: timeline, root cause, impact, and remediation. Post-mortems are blameless — we look for system failures, not human errors."

---

## 12. THIRD-PARTY API FAILURE HANDLING

### Complete Script: "How do you handle third-party API failures?"

> "External dependencies are the most common source of production incidents. We treat them as unreliable and build resilience in:
>
> **Polly Resilience Policies:**
>
> **Retry:** For transient failures (network blip, 503), we retry with exponential backoff and jitter:
> ```csharp
> builder.Services.AddHttpClient<IPaymentGateway, PaymentGatewayClient>()
>     .AddTransientHttpErrorPolicy(policy =>
>         policy.WaitAndRetryAsync(
>             retryCount: 3,
>             sleepDurationProvider: attempt => 
>                 TimeSpan.FromSeconds(Math.Pow(2, attempt)) + 
>                 TimeSpan.FromMilliseconds(Random.Shared.Next(0, 1000))));
> ```
> The jitter (random delay) prevents the **thundering herd problem** — if all instances retry at exactly the same time, you overwhelm the downstream service.
>
> **Circuit Breaker:** If the payment gateway fails 5 times in a row, we open the circuit for 30 seconds. During this time, we fail fast without attempting the call — protecting the gateway from being overwhelmed and protecting our thread pool from being exhausted on waiting connections.
> ```csharp
> .AddTransientHttpErrorPolicy(policy =>
>     policy.CircuitBreakerAsync(
>         handledEventsAllowedBeforeBreaking: 5,
>         durationOfBreak: TimeSpan.FromSeconds(30)));
> ```
>
> **Timeout:** We always set an explicit timeout. The default `HttpClient` timeout in .NET is 100 seconds — way too long for a payment call. We set it to 10 seconds.
>
> **Fallback:** For non-critical external calls (e.g., an address validation API), we have a fallback: if the API is down, we skip validation and flag the record for manual review.
>
> **Idempotency:** For payment APIs, we send an idempotency key with every request. If we retry a timed-out request, the payment provider recognizes the key and returns the original result rather than double-charging.
>
> **Bulkhead:** We use separate `HttpClient` instances (and therefore separate connection pools) for different external APIs. A failure in the payment gateway doesn't exhaust the connection pool used by the email API."

---

## 13. BEHAVIORAL + TECHNICAL HYBRID QUESTIONS

### "Tell me about a complex bug you fixed."

**Framework: STARE**
```
S - Situation     (What was the system? What was failing?)
T - Trigger       (How was the bug discovered? Impact?)
A - Analysis      (How did you diagnose it? What tools?)
R - Resolution    (What was the root cause? What did you change?)
E - Effect        (What was the measurable improvement?)
```

**Sample Answer:**

> "We had a bug in production where certain API requests would randomly return 500 errors about 0.5% of the time, with no pattern in the logs except `InvalidOperationException: A second operation was started on this context before a previous operation completed`.
>
> This is the classic EF Core DbContext thread-safety violation. After analysis, I found a singleton service was injecting `DbContext`, which is scoped. When two concurrent requests hit the singleton, they shared the same DbContext instance, causing concurrent access.
>
> The fix: change the singleton service to scoped, or better, inject `IDbContextFactory<AppDbContext>` and create a new context per operation.
>
> Effect: Zero recurrence of that error. We also added a Roslyn analyzer rule to catch singleton-with-scoped-dependency anti-patterns at compile time."

### "Tell me about a performance optimization you made."

**Framework: IMPACT**
```
I - Initial state      (What was the performance before?)
M - Measurement        (How did you measure the problem?)
P - Problem identified (What was the root cause?)
A - Action taken       (What specifically did you change?)
C - Comparison         (Before vs after metrics)
T - Trade-offs         (What did you give up?)
```

### "Tell me about a design decision you disagreed with."

**This question tests: judgment, professionalism, and communication.**

**What to cover:**
1. What the decision was and why you disagreed (technical reasoning)
2. How you raised your concern (respectfully, with data)
3. What the outcome was (your view was adopted, or you accepted the team decision)
4. What you learned

**Wrong approach:** "I disagreed and was right, they were wrong."  
**Right approach:** "I had a concern, I raised it with evidence, the team considered it, and we made an informed decision — even if it wasn't mine."

---

## 14. THE INFOSYS INTERVIEW STRATEGY

### Round Structure (Typical)

| Round | Focus | Duration | Key Tips |
|-------|-------|----------|----------|
| Technical Round 1 | C#, OOP, DSA basics | 45-60 min | Be precise, write code when asked |
| Technical Round 2 | System design, project deep-dive | 60 min | Use frameworks, draw diagrams |
| HR Round | Behavioral, goals, culture fit | 30 min | STAR format, know "Why Infosys" |

### Infosys-Specific Patterns

**They WILL ask:**
- "Explain SOLID principles with examples from your project"
- "What is the difference between Scoped, Transient, and Singleton?"
- "What is async/await? How does it work internally?"
- "Write a SQL query to find [specific SQL problem]"
- "What design patterns have you used?"

**Infosys traps to prepare for:**
1. "Explain Singleton — is it thread-safe by default?" → They expect you to know about double-checked locking
2. "What happens if you inject a Scoped service into a Singleton?" → Captive dependency problem
3. "Trace through this async code — what is the output order?"
4. "What's wrong with this code?" → Common issues: async void, missing await, null reference

### The "Code on Screen" Scenario

Infosys frequently asks candidates to write code in a shared editor or on a whiteboard. **Key rules:**
1. **Talk while you type.** Explain what you're doing. Interviewers care about your reasoning process.
2. **Start with the skeleton.** Write the method signature, basic structure, then fill in.
3. **Acknowledge what you'd add in production.** "In production, I'd add null checks and logging here."
4. **Don't freeze on syntax.** If you forget an exact method name, say "I'd look this up, but the concept is..."

---

## 15. RESUME LINE-BY-LINE PREPARATION

### For Every Technology You List

Ask yourself these questions. If you can't answer them, remove the technology from your resume OR learn it before the interview:

**For every technology:**
- Why did you use it?
- What problem did it solve?
- What are its limitations?
- What's the alternative?
- Give a specific example of how you used it

### Common Resume Technologies and Questions

| Technology Listed | Common Questions |
|------------------|-----------------|
| **ASP.NET Core** | How does the middleware pipeline work? What is the DI container? |
| **EF Core** | N+1 problem, change tracking, migrations, optimistic concurrency |
| **JWT** | Flow, validation, refresh tokens, invalidation |
| **Redis** | What you cached, eviction policies, distributed lock |
| **CQRS** | Why CQRS? MediatR pipeline? Read model vs write model? |
| **Repository Pattern** | Advantages? Disadvantages? Generic vs specific repository? |
| **Serilog** | Sinks, enrichers, structured logging, sensitive data |
| **Polly** | Which policies? Retry vs circuit breaker? When do you use each? |
| **Docker/Kubernetes** | Dockerfile structure, health checks, resource limits |
| **Azure/AWS** | Which services? Why? How do they integrate? |
| **SQL Server** | Indexes, execution plans, window functions, isolation levels |

---

## 16. COMMON CANDIDATE MISTAKES

### Mistake 1: Answering "I" When It Was "We"

**Problem:** You say "I built the entire system." Interviewer asks a detail about part of the system and you can't answer it.

**Fix:** Be honest about what YOU specifically owned. "My responsibility was X. The Y component was built by my colleague, though I understand the high-level design."

### Mistake 2: Not Knowing Trade-offs

**Problem:** "We used Repository pattern." Interviewer: "What are the disadvantages of Repository?" Candidate: "..."

**Fix:** For every pattern/technology you used, know: what problem it solves, what it DOESN'T solve, and what the cost is.

### Mistake 3: Over-Architecturing Non-Existent Complexity

**Problem:** Describing a CRUD app as if it's a complex distributed system.

**Fix:** Be honest about the scale and complexity of your project. Show maturity: "The system was relatively straightforward — it didn't need microservices. We kept it as a monolith intentionally."

### Mistake 4: Memorized Definitions Without Application

**Problem:** "SOLID stands for Single Responsibility, Open-Closed, Liskov..." — recited from memory with no connection to actual code.

**Fix:** Always connect to code. "The Single Responsibility principle — in my project, I violated it initially by having the Order entity handle both business logic AND persistence. I refactored by moving persistence to the repository and keeping the entity as a pure domain model."

### Mistake 5: Freezing on Follow-Ups

**Problem:** You answer the base question well, but freeze when the interviewer goes one level deeper.

**Fix:** Practice saying the honest boundary response: "That's at the edge of my hands-on experience. Based on what I know, I'd reason that [logical extension], but I'd want to verify this."

### Mistake 6: Not Asking Clarifying Questions in Design Problems

**Problem:** Interviewer says "Design a URL shortener." You immediately start building.

**Fix:** Ask: "What's the expected scale? Do we need analytics? Is this read-heavy or write-heavy? Do we need custom short codes?" Asking clarifying questions is itself a positive signal.

---

## QUICK REFERENCE: ANSWER TEMPLATES

| Question Type | Template |
|--------------|----------|
| "Why did you choose X?" | Problem → Options → Decision → Trade-offs |
| "How does X work?" | Core concept → Implementation → Edge cases |
| "Tell me about a time..." | Situation → Task → Action → Result (STAR) |
| "Design X" | Clarify → Estimate scale → Core entities → API → Scale → Monitor |
| "What's wrong with this code?" | Read aloud → Spot issue → Explain why → Fix it |
| "How do you handle X?" | We identify → We measure → We apply [specific tool] → We validate |

---

*Volume 11 Complete — Proceed to Volume 12 for the 1000+ Question Bank*

---
**Word Count:** ~5,200 words  
**Next:** Volume 12 — Interview Question Bank (1000+ Questions)
