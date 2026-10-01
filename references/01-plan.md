# Plan phase: Layers 1–2

## Contents
- Layer 1: System design
- Layer 2: System architecture

---

## Layer 1: System design

### What it is
The "what and how much" of the product before any code: who the users are, what they do, how much data and traffic to expect, what must never break, and what trade-offs are being made.

### What agents get wrong
- Jump straight to code and invent requirements nobody agreed to.
- Design for a toy (one user, no concurrency) or wildly over-engineer (microservices for 10 users).
- Ignore multi-tenancy: how one customer's data stays separate from another's.

### Decisions the human owns
- Who the customer is: individuals, teams, or companies. This decides the whole data model.
- Pricing model: free tier, per seat, usage-based.
- Realistic load for year one: users, requests per day, data size.
- Non-negotiables: uptime, data privacy, compliance (GDPR, DPDP Act in India, HIPAA, etc.).

### Prompts

**1a. Requirements interview (use before anything else)**
```
I'm building a SaaS: [one paragraph description].
Don't write any code. Act as a senior product engineer and interview me.
Ask me questions, one group at a time, about:
- users and roles
- core workflows (the 3–5 things users actually do)
- tenancy (individual vs team vs organisation accounts)
- data we store and how sensitive it is
- expected scale in year one
- billing and plans
- compliance or regional requirements
- integrations with other services
After I answer, write /docs/system-design.md containing: problem statement,
user roles, core user flows, functional requirements, non-functional
requirements (performance, availability, security), out-of-scope items,
and open questions.
```
In advisor mode, you can run this interview yourself in chat and produce `system-design.md` directly.

**1b. Stress-test the design**
```
Read /docs/system-design.md. Act as a skeptical staff engineer reviewing it.
Identify: missing requirements, ambiguous flows, scaling risks, security and
privacy risks, and anything over-engineered for our stage. Rank issues by
severity. Suggest the simplest design that meets the requirements.
Don't edit the file; give me the review as a list.
```

**1c. Capacity back-of-envelope**
```
Based on /docs/system-design.md, estimate for year one: requests per second
at peak, database size, storage size for uploaded files, and background job
volume. Show your math. Tell me which component becomes the bottleneck first.
```

### Verify
- [ ] `/docs/system-design.md` exists and every line has been read.
- [ ] Tenancy model is written down explicitly.
- [ ] There's an "out of scope" section. Without one, the agent keeps adding features.

---

## Layer 2: System architecture

### What it is
The concrete building blocks: which services exist, how they talk, where data lives, what runs in the background, and which third-party services are relied on.

### What agents get wrong
- Pick whatever stack is most common in training data, not what fits.
- Add dependencies freely (three date libraries, two HTTP clients).
- Mix concerns: business logic in UI components, DB calls in route handlers everywhere.
- Forget background jobs (emails, webhooks, exports) and do everything inside a request.

### Decisions the human owns
- Monolith vs services. For almost every early SaaS: a modular monolith.
- Managed services vs self-hosted.
- Language and framework, based on what they can debug at 2 a.m.

### Prompts

**2a. Architecture proposal**
```
Read /docs/system-design.md. Propose an architecture for this SaaS at our
current stage. Optimise for: a solo/small team, low ops burden, and easy
debugging. Prefer a modular monolith unless there's a strong reason not to.
Give me:
1. Component diagram (as a Mermaid diagram)
2. Stack choice for each layer, with one alternative and why you rejected it
3. Folder structure for the repo
4. Where background jobs run and how
5. List of third-party services, with cost at our scale
6. The 3 riskiest decisions in this architecture
Write it to /docs/architecture.md. Don't create any code yet.
```

**2b. Decision records**
```
For each major decision in /docs/architecture.md, create an Architecture
Decision Record in /docs/decisions/ named NNNN-short-title.md, with:
Context, Decision, Alternatives considered, Consequences.
Then add a summary of the stack and folder structure to CLAUDE.md
(and AGENTS.md) under "Stack" and "Architecture".
```

**2c. Scaffold**
```
Scaffold the project exactly as described in /docs/architecture.md.
Include: folder structure, TypeScript strict mode (or equivalent), linter,
formatter, .env.example with every variable documented, a README with setup
steps, and a health-check endpoint at /api/health.
Don't build features. Run the dev server and the health check to prove it works.
```

### Verify
- [ ] Every box in the architecture diagram can be explained by the human.
- [ ] Every dependency in package.json (or equivalent) has a reason.
- [ ] Business logic lives in a dedicated layer (e.g. `/services` or `/domain`), not in UI or route files.
