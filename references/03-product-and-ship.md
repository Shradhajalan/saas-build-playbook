# Product & Ship phase: Layers 6–9

## Contents
- Layer 6: Frontend
- Layer 7: CI/CD & version control
- Layer 8: Testing
- Layer 9: Hosting & cloud

---

## Layer 6: Frontend

### What it is
Everything the user sees and touches: pages, forms, loading states, errors, responsiveness, accessibility.

### What agents get wrong
- Only build the happy path. No loading, empty or error states.
- Generic, templated look that screams "AI made this".
- Poor accessibility: missing labels, no keyboard navigation, low contrast.
- Giant components, duplicated UI code, inconsistent spacing.
- Leaking secrets into client-side code.

### Decisions the human owns
- Design direction and brand. Give the agent references and screenshots.
- Component library (shadcn/ui, Radix, MUI, etc.).
- Which screens actually matter for launch.

### Prompts

**6a. Design system first**
```
Before building screens, create a small design system:
- color tokens (with dark mode), typography scale, spacing scale, radii
- base components: Button, Input, Select, Modal, Toast, Table, EmptyState,
  ErrorState, Skeleton
- a /design page that shows every component in every state
Style direction: [describe, or reference 2–3 products you like].
Avoid generic template look. Use these components everywhere from now on.
```

**6b. Build a screen properly**
```
Build the [screen name] screen using our design system and the existing API.
Every data-driven view must handle 4 states: loading (skeletons),
empty (helpful message + primary action), error (message + retry),
and success. Forms need: client-side validation matching the server rules,
disabled submit while pending, inline field errors, and success feedback.
Must work at 375px mobile width and on desktop.
```

**6c. Accessibility pass**
```
Audit the frontend for accessibility (WCAG 2.1 AA): labels on all inputs,
keyboard navigation for every interactive element, visible focus states,
colour contrast, alt text, correct heading order, and ARIA only where needed.
Fix issues and add an automated accessibility check (e.g. axe) to our tests.
```

**6d. Frontend secrets check**
```
Search the frontend bundle and all client-side code for API keys, secrets
or private URLs. Only variables intended to be public should be exposed
to the browser. Report what you found and fix it.
```

### Verify
- [ ] Turn off Wi-Fi and use the app. Every screen shows a sensible error.
- [ ] Use the whole app with only a keyboard.
- [ ] Open it on an actual phone.

---

## Layer 7: CI/CD & version control

### What it is
**Version control** = Git: every change tracked, reviewable, reversible. **CI** = every push runs lint, typecheck, tests automatically. **CD** = passing code deploys automatically to staging/production.

### What agents get wrong
- Huge commits mixing ten unrelated changes.
- Committing `.env` files or secrets.
- No CI, so broken code reaches production.
- Running database migrations manually and inconsistently.

### Decisions the human owns
- Branching: `main` is always deployable; feature branches + pull requests.
- Environments: at minimum local, staging (preview), production.
- Whether production deploys are automatic or need approval.

### Prompts

**7a. Git hygiene**
```
Set up version control hygiene:
- .gitignore that covers env files, build output, OS files, editor files
- pre-commit hook that runs lint and format on staged files
- a secret scanner in pre-commit (e.g. gitleaks)
- commit message convention (Conventional Commits) documented in CLAUDE.md
Then scan the full git history for any committed secrets and tell me what you find.
```

**7b. CI pipeline**
```
Create a GitHub Actions CI workflow that runs on every pull request and
push to main:
install (with caching) → lint → typecheck → unit tests → integration tests
against a real Postgres service container → build.
Fail fast. Keep it under 10 minutes. Add a status badge to the README.
```

**7c. CD pipeline**
```
Set up deployments:
- every PR gets a preview deployment
- merge to main deploys to staging automatically
- production deploy requires a manual approval step
- database migrations run automatically as part of deploy, before the new
  code goes live, and the deploy fails if a migration fails
- document how to roll back to the previous version in /docs/runbooks/rollback.md
```

**7d. Working with the agent in Git**
```
From now on: create a new branch for each task, make small commits with
clear messages, and open a pull request with a description of what changed,
why, how to test it, and any risks. Never push directly to main.
```

### Verify
- [ ] Open a PR with a deliberately failing test. CI blocks it.
- [ ] A deployment has been rolled back once, on purpose, to practise.
- [ ] `git log` reads like a story, not "fix", "fix2", "final fix".

---

## Layer 8: Testing

### What it is
Automated checks that prove the app does what it should and keeps doing it as code changes. Unit (one function), integration (API + database), end-to-end (a real browser clicking through).

### What agents get wrong
- Write tests that test the mocks, not real behaviour.
- **Change the test to make it pass instead of fixing the code.** Watch for this constantly.
- Skip or delete failing tests quietly.
- Only test happy paths.

### Decisions the human owns
- What must never break: signup, login, payments, core workflow, data isolation. These get end-to-end tests.
- Coverage target: 70–80% on business logic is sensible; 100% is rarely worth it.

### Prompts

**8a. Test strategy**
```
Write /docs/testing.md with our test strategy:
- unit tests for /services business logic
- integration tests for every API endpoint using a real test database
- end-to-end tests (Playwright) for these critical flows: signup, login,
  [core workflow], upgrade plan, invite teammate
- which things are mocked (external APIs only) and which are never mocked
  (our database, our auth checks)
Set up the tooling for all three levels.
```

**8b. Tests for existing code**
```
Write tests for [module]. For each function, cover: happy path, invalid
input, boundary values, permission denied, and cross-tenant access.
Rules: do not modify the code under test. If a test reveals a bug, stop
and report it to me rather than changing the test to pass.
```

**8c. Test-driven bug fixing**
```
Bug: [describe the bug and how to reproduce].
1. First write a failing test that reproduces it. Show me it failing.
2. Then fix the code.
3. Show me the test passing and the full suite still green.
```

**8d. Test quality review**
```
Review our test suite critically. Find: tests that would still pass if the
feature were broken, tests that only assert on mocks, skipped tests,
flaky tests, and important paths with no coverage. Give me a prioritised list.
```

### Verify
- [ ] Deliberately break a line of business logic. At least one test fails.
- [ ] grep for `.skip`, `xit`, `@pytest.mark.skip`. Every skip has a reason.
- [ ] After each agent session, check the diff for changes to test files nobody asked for.

---

## Layer 9: Hosting & cloud

### What it is
Where the app runs in production: frontend hosting, backend servers, database, file storage, DNS, domains, SSL, environment variables.

### What agents get wrong
- Suggest complex setups (Kubernetes, Terraform everything) way too early.
- Use the same database for staging and production.
- Hard-code URLs and config that differ between environments.
- Forget cost → surprise bill.

### Decisions the human owns
- Platform: Vercel, Netlify, Railway, Render, Fly.io, or AWS/GCP/Azure. Managed platforms are usually right early.
- Region: close to users (and for data residency rules).
- Budget ceiling and billing alerts.

### Prompts

**9a. Hosting plan**
```
Recommend a hosting setup for our architecture in /docs/architecture.md.
Constraints: small team, minimal ops, users mostly in [region],
budget around [amount]/month at launch.
For each component (frontend, backend, database, storage, background jobs,
email), give provider, plan, estimated monthly cost at launch and at 10x
users, and what we'd need to change at 10x.
Write it to /docs/hosting.md.
```

**9b. Environment configuration**
```
Make the app fully configurable by environment variables. Produce:
- .env.example listing every variable with a comment explaining it
- runtime validation at startup that fails loudly if a required
  variable is missing or malformed
- separate values for local, staging, production
- no URLs, keys or IDs hard-coded anywhere
List every place you changed.
```

**9c. Production readiness**
```
Prepare the app for production deployment on [platform]:
- custom domain and HTTPS
- security headers
- separate production database with automated backups enabled
- health check endpoint wired to the platform
- graceful shutdown so in-flight requests finish during deploys
Then write /docs/runbooks/deploy.md with step-by-step first-deploy instructions.
```

### Verify
- [ ] Staging and production use different databases and different API keys.
- [ ] Billing alerts are set on every provider.
- [ ] Removing a required env variable makes the app refuse to start, with a clear message.
