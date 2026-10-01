# Operate & Grow phase: Layers 13–15, and beyond

## Contents
- Layer 13: Error tracking & logs
- Layer 14: Monitoring & alerts
- Layer 15: Scaling
- Beyond the 15 layers

---

## Layer 13: Error tracking & logs

### What it is
**Error tracking** captures every crash with stack trace, user and context (e.g. Sentry). **Logs** are a searchable record of what the system did.

### What agents get wrong
- `console.log` everywhere, no structure, no levels.
- Swallowing errors with empty catch blocks, so failures are silent.
- Logging sensitive data.
- No way to trace one user's request through the system.

### Decisions the human owns
- Error tracking tool (Sentry, Rollbar, Highlight, etc.).
- Log destination and retention (platform logs, Axiom, Better Stack, Datadog, etc.).

### Prompts

**13a. Structured logging**
```
Replace all console.log/print calls with a structured logger (JSON output,
levels: debug, info, warn, error).
- Every request gets a request ID, included in every log line and returned
  in a response header
- Every log line includes user_id and organization_id when available
- Automatically redact fields named password, token, secret, authorization,
  and any fields in [list of personal data]
- debug level is off in production
Show me an example log line for a request.
```

**13b. Error tracking**
```
Integrate [Sentry] on both frontend and backend.
- Attach user_id, organization_id and request ID to every error (no emails
  or personal data unless I approve)
- Upload source maps so stack traces are readable, but don't serve them publicly
- Separate environments (staging vs production)
- Add a test route, only in staging, that throws an error so I can verify it arrives
```

**13c. Error handling cleanup**
```
Find every empty or overly broad catch block, every ignored promise
rejection, and every place an error is caught but not logged or re-thrown.
Fix them so errors are either handled meaningfully or reported. Make sure
users see a friendly message while the full detail goes to error tracking.
```

### Verify
- [ ] An error triggered in staging appears in the tracker with a readable stack trace within a minute.
- [ ] A request ID from a response header finds every log line for that request.
- [ ] Searching logs for "password" returns zero results.

---

## Layer 14: Monitoring & alerts

### What it is
Knowing the app is healthy before users say it isn't: uptime checks, performance metrics, business metrics, and alerts that wake someone only when it matters.

### What agents get wrong
- Nothing is set up, because nothing is visible until it breaks.
- Too many noisy alerts, so all of them get ignored.
- Monitoring only the server, not whether users can do the core thing.

### Decisions the human owns
- What counts as "down" or "degraded".
- Who gets alerted, how (email, Slack, SMS, phone) and when.
- Service level target (e.g. 99.9% uptime).

### Prompts

**14a. Monitoring plan**
```
Write /docs/monitoring.md defining:
- uptime checks for: homepage, login, /api/health, and [core workflow endpoint]
- key technical metrics: error rate, p95 latency per endpoint, database
  connections, background job queue depth and failure rate
- key business metrics: signups, active orgs, successful payments, failed payments
- alert rules: what threshold, how long, severity, and who is notified
Keep alerts to only things that need a human to act.
```

**14b. Health checks**
```
Upgrade /api/health to a real health check: verify database connectivity,
cache connectivity, and background job system, with timeouts, returning
component-level status. Keep a lightweight /api/health/live for the load
balancer that only confirms the process is up.
```

**14c. Alerts and runbooks**
```
For each alert in /docs/monitoring.md, write a runbook in /docs/runbooks/
with: what the alert means, how to check impact, likely causes, step-by-step
fixes, and when to escalate. Link each alert to its runbook.
```

**14d. Status and incident process**
```
Set up a simple public status page and write /docs/runbooks/incident.md:
how to declare an incident, how to communicate with customers, and a
template for a short post-incident review.
```

### Verify
- [ ] Stopping the database in staging gets an alert to a human within the defined time.
- [ ] Every alert has a runbook.
- [ ] The dashboard is looked at at least once a week.

---

## Layer 15: Scaling

### What it is
Handling more users, data and traffic without slowing down, falling over, or costs exploding.

### What agents get wrong
- Over-engineer for scale that doesn't exist (sharding, microservices, Kubernetes on day one).
- Or ignore the basics that actually break first: missing indexes, N+1 queries, no connection pooling, synchronous heavy work.

### Decisions the human owns
- When to scale. Measure first, then optimise the actual bottleneck.
- Infrastructure budget as they grow.

### Prompts

**15a. Load test**
```
Write load tests (e.g. k6) for our 5 most important endpoints and the
core user flow. Simulate [N] concurrent users ramping up over 10 minutes
against staging. Report p50/p95/p99 latency, error rate, and the point
where performance degrades. Identify the bottleneck component.
```

**15b. Scaling readiness review**
```
Review the app for scaling readiness:
- is the app stateless (no in-memory sessions, uploads, or rate limit counters)
  so we can run multiple instances?
- database connection pooling configured correctly for serverless / multiple instances?
- slow queries and missing indexes (use EXPLAIN on the top queries)
- heavy work that should be in background jobs
- unbounded queries or list endpoints without pagination
- large payloads
Rank by what will break first as traffic grows 10x and 100x.
```

**15c. Targeted optimisation**
```
Our bottleneck is [X, from load test or monitoring]. Propose 3 options to fix
it, from simplest to most complex, with cost and effort for each.
Implement the simplest one that meets a target of [p95 < 300ms at N users].
Re-run the load test and show before/after numbers.
```

**15d. Cost at scale**
```
Using /docs/hosting.md and our current usage, model monthly infrastructure
cost at 10x and 100x users. Identify the most expensive component at each
level and suggest ways to reduce it.
```

### Verify
- [ ] A load test result with real numbers exists, not a guess.
- [ ] Two backend instances run at once without anything breaking.
- [ ] Every optimisation has before/after numbers.

---

## Beyond the 15 layers

Once the fifteen are solid, these come next:

- **Emails:** transactional provider, templates, SPF/DKIM/DMARC so mail doesn't land in spam.
- **Analytics:** product analytics to see what users actually do.
- **Feature flags:** release to a few users first, turn off instantly if it breaks.
- **Legal:** terms, privacy policy, cookie consent, data processing agreements.
- **Admin panel:** internal tools to support customers safely, with audit logs.
- **Audit logs:** who did what in each organisation (enterprise buyers ask for this).
- **Onboarding:** first-run experience, sample data, docs, help center.
- **Internationalisation:** languages, currencies, time zones.
- **Disaster recovery:** a tested plan for "our main provider is down" or "data was deleted".

There are no pre-written prompts for these. Apply the same pattern to write them on request: what it is, what agents get wrong, decisions owned, prompts (plan first), verify checklist. Always include org scoping and server-side enforcement where relevant.
