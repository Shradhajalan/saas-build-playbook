# Harden phase: Layers 10–12

## Contents
- Layer 10: Security
- Layer 11: Rate limiting
- Layer 12: Caching & CDN

---

## Layer 10: Security

### What it is
Protecting users' data and the system from attacks: injection, XSS, broken access control, leaked secrets, vulnerable dependencies, and more.

### What agents get wrong
- Build what's asked without thinking like an attacker.
- Trust user input in queries, HTML, file paths and redirects.
- Log sensitive data (passwords, tokens, personal info).
- Add outdated or vulnerable packages.
- Leave debug endpoints and verbose errors on in production.

### Decisions the human owns
- What data is sensitive and how it's protected.
- Compliance obligations for their market.
- Whether to get an external penetration test before handling sensitive data.

### Prompts

**10a. Threat model**
```
Create a threat model for our app in /docs/security/threat-model.md.
List our assets (data, accounts, payments), entry points (every endpoint,
upload, webhook, form), and for each entry point the relevant threats from
the OWASP Top 10. Rate likelihood and impact, and list mitigations we have
and mitigations we're missing.
```

**10b. Security review (run before every release)**
```
Act as a security reviewer. Audit the codebase for:
- broken access control and missing org scoping
- SQL/NoSQL injection and unsafe raw queries
- XSS (unsafe HTML rendering)
- CSRF on state-changing requests
- open redirects and SSRF (server fetching user-supplied URLs)
- insecure file uploads
- secrets in code or logs, sensitive data in logs
- missing security headers (CSP, HSTS, X-Frame-Options, etc.)
- verbose error messages leaking internals
- insecure cookie settings
Report each finding with file, line, severity, exploit scenario and fix.
Do not fix anything yet.
```
Run this in a fresh session or a separate reviewer subagent, not the one that wrote the code. Good candidate for a saved slash command (`.claude/commands/security-review.md`).

**10c. Dependency security**
```
Run a dependency vulnerability audit. List vulnerable packages, severity,
and whether we actually use the vulnerable code path. Upgrade what's safe,
and explain anything you can't upgrade. Then set up automated dependency
update PRs (e.g. Dependabot or Renovate) and add the audit to CI.
```

**10d. Data protection**
```
Implement: encryption of sensitive fields at rest [list fields], redaction
of passwords, tokens, and personal data from all logs, account deletion
that removes or anonymises the user's data, and a data export feature
so users can download their data.
```

### Verify
- [ ] Review prompt was run by a fresh session or reviewer subagent.
- [ ] Paste `<script>alert(1)</script>` into every text field. Nothing pops up.
- [ ] Logs contain no email addresses, tokens or passwords.

AI security reviews catch a lot, but they are not a substitute for a professional audit when handling payments, health, financial or other sensitive data.

---

## Layer 11: Rate limiting

### What it is
Capping how many requests someone can make in a time window. Protects against brute-force logins, scrapers, abuse, runaway scripts and surprise bills (especially with paid AI APIs).

### What agents get wrong
- Don't add it unless asked.
- Use in-memory counters that reset on deploy and don't work across multiple servers.
- Apply one global limit instead of per route and per plan.

### Decisions the human owns
- Limits per route: login and signup strictest, expensive endpoints next, reads loosest.
- Limits per plan (free vs paid).
- What happens at the limit: block, queue, or charge overage.

### Prompts

**11a. Rate limiting setup**
```
Implement rate limiting using a shared store (e.g. Redis / Upstash) so it
works across multiple server instances.
Limits:
- login, signup, password reset: [5 per 15 min per IP and per email]
- expensive endpoints [list, e.g. AI generation, exports]: [N per minute per org]
- general API: [N per minute per user]
- public/unauthenticated endpoints: [N per minute per IP]
Return 429 with a Retry-After header and our standard error format.
Make limits configurable per plan.
Write tests that exceed each limit.
```

**11b. Cost protection for AI or paid APIs**
```
We call [paid API, e.g. an LLM API]. Add:
- per-org daily and monthly usage quotas tied to plan
- usage tracking stored in the database
- a hard global spending cap that disables the feature if exceeded
- an alert to me when any org hits 80% of quota or the global cap nears
- a usage page in the UI so customers see their consumption
```

**11c. Abuse review**
```
Review our public surface for abuse risks: signup spam, email sending
abuse (invites, password resets), enumeration of users via error messages,
scraping of public pages, and webhook endpoints that could be flooded.
Propose protections for each, including bot protection on signup.
```

### Verify
- [ ] A quick loop hitting login 20 times gets blocked.
- [ ] Limits still work after a redeploy.
- [ ] "Wrong password" and "no such user" return the same message.

---

## Layer 12: Caching & CDN

### What it is
**Caching** = storing results so they aren't recomputed or refetched every time. **CDN** = serving static files (JS, CSS, images) from servers close to the user.

### What agents get wrong
- Cache too aggressively, so users see stale or, worse, another tenant's data.
- Cache nothing, so every page hits the database.
- Forget invalidation when data changes.
- Serve unoptimised, huge images.

### Decisions the human owns
- What can be stale and for how long.
- Cache store (often the hosting platform's CDN plus Redis for app data).

### Prompts

**12a. Caching strategy**
```
Analyse our app and propose a caching strategy in /docs/caching.md.
For each candidate (static assets, public marketing pages, API responses,
expensive queries, third-party API results), specify: where it's cached
(browser, CDN, server, Redis), TTL, cache key, and how it's invalidated
when data changes.
Rule: any cache of tenant data must include organization_id in the key,
and authenticated responses must never be cached at a shared CDN layer.
```

**12b. Implement CDN and static assets**
```
Configure static assets to be served via CDN with long cache lifetimes and
content-hashed filenames. Optimise images (modern formats, responsive sizes,
lazy loading). Set correct Cache-Control headers for: static assets,
public pages, and authenticated API responses (private, no-store where needed).
Show me the headers for one example of each.
```

**12c. Application caching**
```
Add Redis caching for these slow operations: [list]. Use a cache-aside
pattern with keys that include organization_id. Invalidate on every write
that affects the data. Add a way to bypass the cache for debugging.
Write tests proving that: cache is invalidated on update, and two different
orgs never receive each other's cached data.
```

### Verify
- [ ] Two users from different orgs in two browsers never see each other's data, even after refreshes.
- [ ] Updating a record shows the new value immediately (or within the agreed TTL).
- [ ] Lighthouse audit run on the main pages.
