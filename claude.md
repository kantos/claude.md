# CLAUDE.md — Project Configuration & Engineering Standards

> **How to use this file**
>
> **Starting out:** Fill in every section marked with `[PLACEHOLDER]`. Leave everything else as-is — Claude will use these rules automatically on every task. The more detail you add to the placeholder sections, the better Claude can help you.
>
> **As your project grows:** This file is yours to update. Whenever you make a decision — about hosting, a tool, a feature, or anything technical — tell Claude to update this file to reflect it. Examples of things to say:
> - *"Update CLAUDE.md to reflect my decision to deploy to Cloudflare instead of Vercel."*
> - *"Update CLAUDE.md — we are using Supabase for the database and auth."*
> - *"Update CLAUDE.md — we've decided to drop the mobile web requirement, desktop only."*
> - *"Update CLAUDE.md to add SendGrid as our approved email provider."*
>
> Think of this file as the single source of truth for your project. If a decision isn't written here, Claude won't know about it in future sessions. When in doubt, update the file.
>
> **Using Claude Code (recommended for building):** Claude Code reads this file automatically at the start of every session, so your rules and decisions are always applied without you having to repeat yourself. If you are using the Claude.ai website instead, paste or upload this file at the start of each conversation.

---

## 1. Project Overview

### What I Am Building
```
[PLACEHOLDER — Describe your web app in plain English. Examples:
  - "A booking platform where dog owners can find and pay local dog sitters."
  - "An internal dashboard for my restaurant chain to track daily sales per location."
  - "A marketplace where freelance translators can list services and get hired."
Be as specific as possible: who uses it, what problem it solves, how they use it.]
```

### Target Users
```
[PLACEHOLDER — Who will use this app?
  - "Small business owners with no tech background, aged 35–60."
  - "HR managers at mid-size companies."
  - "Teenagers in Spain who want to trade collectibles."
Include approximate technical level of users (beginner / intermediate / advanced).]
```

### Core Features (MVP)
```
[PLACEHOLDER — List the must-have features for the first version. Examples:
  1. User registration and login with email/password.
  2. A dashboard showing key metrics.
  3. Ability to upload and manage documents.
  4. Stripe payments for subscriptions.
  5. Email notifications on key events.
Keep this list focused. You can add "Nice to have" separately below.]
```

### Nice-to-Have Features (Post-MVP)
```
[PLACEHOLDER — Features you want eventually but not on day one.]
```

### What This App Must NOT Do
```
[PLACEHOLDER — Any explicit constraints or out-of-scope items. Examples:
  - "Must not store any health or medical data (HIPAA not in scope)."
  - "Must not have a mobile native app — web only."
  - "Must not support payments in crypto."]
```

### Technology Preferences
```
[PLACEHOLDER — If you have already decided on any tools or platforms, state them here.
Claude will respect these decisions and update this file as more decisions are made. Examples:
  - "Deploy to Cloudflare Pages."
  - "Use Supabase for the database and auth."
  - "Use Resend for email."
  - "Use Stripe for payments."
Leave blank if you have no preferences — Claude will choose sensible defaults.]
```

---

## 2. Tech Stack

> Claude will make all technology decisions. The rules below are non-negotiable constraints Claude must respect when choosing tools.

### Approved Stack (Claude will decide specifics within these boundaries)
- **Language**: TypeScript (strict mode) everywhere — frontend and backend.
- **Frontend**: React with a file-based routing framework (e.g., Next.js).
- **Backend**: Node.js — REST or tRPC as Claude sees fit.
- **Database**: PostgreSQL (primary). Redis only if caching is genuinely needed.
- **Auth**: A managed auth provider (e.g., Clerk, Auth.js, Supabase Auth). Never roll custom auth.
- **Hosting**: Vercel (frontend/edge), Railway or Render (backend/DB) unless otherwise specified.
- **CSS**: Tailwind CSS. No CSS-in-JS libraries.
- **Forms**: React Hook Form + Zod for validation.

### Third-Party Libraries Policy — STRICT MINIMUM
Claude MUST follow these rules before adding any external dependency:

1. **Justify every dependency.** Before adding a library, state: what it does, why the native platform cannot do it, and what the bundle/maintenance cost is.
2. **Prefer platform APIs first.** Use Web APIs, Node built-ins, and framework built-ins before reaching for npm.
3. **No library for trivial tasks.** Do not install a library to format a date, generate a UUID, debounce a function, or deep-clone an object. Write the utility function instead.
4. **Audit transitive dependencies.** If a library brings in 50 sub-packages, look for a lighter alternative.
5. **One library per problem.** Never install two libraries that solve the same problem.
6. **Always use the latest stable version.** When adding or upgrading any library, tool, or framework, always install the latest stable release. Never pin to an old version without a documented reason. Keep dependencies up to date — running behind on versions is a security and compatibility risk.
7. **No abandonware.** Libraries must have been updated within the last 12 months and have >100k weekly npm downloads unless there is a compelling reason.

---

## 3. Architecture Principles

Claude must follow these architectural patterns on every task:

- **12-Factor App** principles: config via environment variables, stateless processes, explicit dependencies.
- **API-first**: All business logic lives in the API layer. The frontend is a thin UI layer only.
- **Thin controllers, fat services**: Route handlers/controllers do nothing except validate input and call a service. Business logic lives in service classes/functions.
- **No logic in the database**: No stored procedures or triggers. Migrations only change schema.
- **Feature folders**: Code is organised by feature, not by type (no giant `utils/` or `helpers/` folders).
- **Explicit over implicit**: No magic. Every import is explicit. No barrel files that re-export everything.
- **Right-size the architecture**: Do not introduce monorepos, microservices, or complex infrastructure patterns unless the project explicitly requires them. Start simple and scale when there is a proven need.

---

## 4. Git Hygiene — NON-NEGOTIABLE

### Secrets — Never Commit Them
- **No secret ever touches git.** API keys, passwords, database URLs, tokens, and private certificates must never appear in any committed file — not even in old commits.
- All secrets go in `.env.local` (local dev) or the hosting platform's secret manager (staging/production).
- The only `.env` file committed to the repo is `.env.example`, which contains placeholder values and comments — never real credentials.
- If a secret is ever accidentally committed, treat it as compromised immediately: rotate it, then scrub the git history.
- Claude must check before every commit that no secret is being introduced.

### `.gitignore` — Always Up to Date
- A comprehensive `.gitignore` must exist from the very first commit.
- It must always cover at minimum: `.env.local`, `.env*.local`, `node_modules/`, build output directories (`dist/`, `.next/`, `out/`), OS files (`.DS_Store`, `Thumbs.db`), editor files (`.vscode/`, `.idea/`), and any log or cache files.
- Whenever a new tool, framework, or output directory is added to the project, the `.gitignore` must be updated in the same commit.
- Claude must never commit a file that belongs in `.gitignore`.

### Branching Strategy
- **`main` is always deployable.** No direct commits to `main`. Ever.
- Every piece of work — features, bug fixes, experiments, dependency upgrades — happens on its own branch.
- Branch naming convention: `feat/short-description`, `fix/short-description`, `chore/short-description`.
- Branches are merged into `main` via a Pull Request only, after CI passes and (where applicable) a review is completed.
- Delete branches after merging — keep the repo clean.

---

## 5. Code Quality Standards — NON-NEGOTIABLE

These rules apply to every single file Claude writes or modifies.

### TypeScript
- `strict: true` in `tsconfig.json`. No exceptions.
- No `any` type. Use `unknown` and narrow it explicitly.
- All function parameters and return types must be explicitly typed.
- Use `type` for data shapes; use `interface` only for objects that will be extended.
- Zod schemas are the single source of truth for all external data validation (API inputs, env vars, form data).

### Testing — Minimum 80% Code Coverage
Claude must write tests as part of every feature, not as an afterthought.

- **Framework**: Vitest (unit & integration), Playwright (end-to-end).
- **Coverage gate**: 80% line, branch, function, and statement coverage. CI must fail below this threshold.
- **Unit tests**: Every service function, utility, and data transformer must have unit tests.
- **Integration tests**: Every API endpoint must have at least one happy-path and one error-path integration test.
- **E2E tests**: Every critical user flow (sign-up, login, core action, payment) must have an E2E test.
- **Test naming convention**: `it("should [expected behaviour] when [condition]", ...)`.
- **No testing implementation details**: Test behaviour and outputs, not internal function calls.
- **Mocking rules**: Only mock external I/O (HTTP calls, email, third-party APIs). Never mock the module under test.
- **Test data**: Use factories (e.g., `@anatine/zod-mock` or hand-written builders), never hard-coded magic strings.

### Security Testing
Security controls must be proven by tests, not just implemented. Claude must write the following security tests for every relevant feature:

- **Auth boundary tests**: every protected route must be tested with no token, an expired token, and a valid token belonging to a different user. All three must be rejected.
- **Authorisation tests**: user A must not be able to read, edit, or delete user B's resources. Test this explicitly for every resource type — do not rely solely on middleware.
- **Input validation tests**: every API endpoint must be tested with missing fields, wrong types, oversized payloads, and malicious strings (SQL injection patterns, script tags). All must be rejected with a 400, never a 500.
- **Rate limiting tests**: verify that endpoints with rate limits actually block requests after the threshold is reached.
- **Webhook integrity tests**: verify that webhook endpoints reject payloads with invalid or missing signatures.

These tests are not optional and count toward the 80% coverage gate. Critical auth and payment paths must be fully covered regardless of the overall percentage.

### Code Style
- ESLint + Prettier. Config committed to the repo. Claude must not disable lint rules inline without a comment explaining why.
- Max file length: 300 lines. If a file exceeds this, refactor into smaller modules.
- Max function length: 40 lines. If longer, extract sub-functions.
- Cyclomatic complexity max: 10 per function.
- No `console.log` in committed code. Use a structured logger (e.g., `pino`).
- All async functions must handle errors — no unhandled promise rejections.
- `TODO` comments must include a GitHub issue number: `// TODO(#42): refactor after API v2`.

### Git Commit Convention
Follow [Conventional Commits](https://www.conventionalcommits.org/) for all commits.

---

## 6. Security — Non-Negotiable Requirements

### OWASP Top 10
All code must comply with the current [OWASP Top 10](https://owasp.org/www-project-top-ten/). Claude knows what this standard requires and must apply the appropriate mitigations for every risk category by default, without being asked.

### GDPR & Data Privacy
If the app collects personal data from users in the EU — which should be assumed by default — GDPR applies. Claude must: collect only the minimum personal data needed for each feature (data minimisation); ensure users can request deletion of their data; never store personal data longer than necessary; flag any new feature that introduces new personal data collection and confirm how it is handled before implementing it. When in doubt, treat the data as personal and apply the stricter standard.

### Additional Web Security Headers
Claude must configure the following HTTP headers on every response:
```
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self';
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

### Input/Output Rules
- Every input is untrusted until validated. No exceptions: API parameters, query strings, request bodies, headers, file uploads, webhook payloads — all must pass through a Zod schema before any business logic touches them.
- Validation must happen server-side. Client-side validation is UX only and must never be relied upon for security.
- All HTML rendered from user data must be escaped. Never use `dangerouslySetInnerHTML` with user data.
- File uploads: validate MIME type server-side (not just by extension), enforce max size, store in object storage (never on the local filesystem), scan with an antivirus API if the file will be shared with other users.

---

## 7. GitHub Pull Request & CI/CD Security Checks

### Required GitHub Actions Checks

Every PR must pass all of the following automated checks before merge. Claude knows how to implement each of these and must set them up when initialising the project.

- **Semgrep** — Static Application Security Testing (SAST). Scans code for security vulnerabilities on every PR. Any finding at ERROR or WARNING severity blocks the merge.
- **TruffleHog** — Secret scanning. Scans the full git history of every PR branch to detect accidentally committed secrets (API keys, passwords, tokens). Any verified finding blocks the merge immediately.
- **SCA (Software Composition Analysis)** — Dependency vulnerability scanning using `npm audit` and OSV Scanner. Fails on any HIGH or CRITICAL CVE in direct or transitive dependencies.
- **Lint** — ESLint must report zero errors.
- **Type check** — TypeScript compiler must report zero errors.
- **Tests** — All tests must pass and coverage must be ≥ 80%.
- **Build** — Production build must succeed.

### Branch Protection Rules (Claude must document these for the repo owner to set in GitHub)
- `main` branch: require PR reviews (min 1 approval), require all status checks to pass, no direct pushes, no force pushes.
- Signed commits: recommended.
- Delete branches after merge: enabled.

---

## 8. Accessibility & Performance

### Accessibility (WCAG 2.1 AA minimum)
- All interactive elements must be keyboard navigable.
- All images must have meaningful `alt` text.
- Colour contrast ratio: minimum 4.5:1 for normal text, 3:1 for large text.
- Use semantic HTML elements (`<nav>`, `<main>`, `<section>`, `<button>`, etc.) correctly.
- Forms must have associated `<label>` elements.
- Use `axe-core` in E2E tests to catch regressions automatically.

### Performance Targets
- Core Web Vitals: LCP < 2.5s, CLS < 0.1, INP < 200ms.
- Bundle size: no single JS chunk > 200 KB (gzipped).
- Images: use `next/image` or equivalent; always serve WebP/AVIF; use lazy loading.
- No layout shift from late-loading fonts: use `font-display: optional` or preload critical fonts.
- API response times: < 200ms for read endpoints, < 500ms for write endpoints (p95).

---

## 9. Logging, Observability & Error Handling

### Logging Tool
Use `pino` as the structured logger. It outputs JSON, is fast, and works well with all major log aggregation platforms (Datadog, Logtail, Grafana Loki, etc.). No `console.log` anywhere in committed code — ever.

### Required Fields on Every Log Line
Every log entry must include: `timestamp`, `level`, `requestId` (a unique ID per HTTP request), `userId` (if the user is authenticated), `service` (name of the app/service), and `message`. This makes it possible to trace exactly what happened and who triggered it.

### Three Distinct Log Streams

**1. Application / Debug Logs**
What they cover: server start/stop, configuration loaded, background jobs running, cache hits/misses, external API calls made, database query timing, unexpected code paths, caught errors with full stack traces.
Purpose: debugging and understanding system behaviour. Not for business analysis.

**2. User Activity Logs (Audit Trail)**
What they cover: every meaningful action a user takes — signed up, logged in, logged out, failed login attempt, changed password, changed email, updated profile, created/edited/deleted any resource, made a payment, invited another user, changed permissions, exported data, deleted their account.
Each entry must record: `userId`, `action`, `resourceType`, `resourceId`, `ipAddress`, `userAgent`, `timestamp`, and `outcome` (success or failure).
Purpose: knowing exactly what every user did and when. Critical for support, compliance, and detecting abuse. These logs must never be deleted and must be stored separately from debug logs.

**3. Security / System Event Logs**
What they cover: authentication failures, repeated failed login attempts, permission denied events, secrets rotation, admin actions, CI/CD deployments, configuration changes, dependency vulnerability alerts.
Purpose: detecting attacks, auditing admin behaviour, and maintaining a chain of evidence if something goes wrong.

### What Must Never Appear in Any Log
- Passwords, in any form.
- Full credit card numbers, CVVs, or bank details.
- Auth tokens, session tokens, or API keys.
- Full request bodies that may contain any of the above.
- Personally identifiable information (PII) beyond what is strictly necessary for the log's purpose (e.g. a user ID is fine; a full name + email + address in every line is not).

If a field might contain sensitive data, redact it before logging: `{ password: "[REDACTED]" }`.

### Error Handling
- **Never expose internal errors to the client.** Return a generic user-facing message; log the full error with stack trace server-side.
- All API errors must follow a consistent shape: `{ error: { code: string, message: string } }`.
- All async functions must handle errors explicitly — no unhandled promise rejections.
- Use a dedicated error monitoring service (Sentry recommended) in production. It captures exceptions automatically, groups them, and alerts on spikes.

### Alerting
Set up alerts for: error rate spike above baseline, repeated authentication failures from the same IP, any security log event, and a health check failure. Claude will configure these when setting up the observability stack.

### Health Check
Expose `GET /health` returning `{ status: "ok", version: string, uptime: number }`. This endpoint must not require authentication and must be monitored externally (e.g. via UptimeRobot or the hosting platform's health checks).

---

## 10. Database & Data Management

- All schema changes via migrations (never manually alter the production database).
- Migration tool: Drizzle ORM or Prisma — Claude will choose and stay consistent.
- Every table must have: `id` (UUID default), `created_at`, `updated_at`.
- Soft deletes (add `deleted_at` column) for any user-facing entity — never hard-delete user data unless legally required.
- Database connection pooling configured appropriately for the hosting environment.
- Never run migrations automatically on app startup in production. Run as a separate deploy step.
- Backup policy: automated daily backups retained for at least 30 days.

---

## 11. Documentation Requirements

Claude must keep the following documentation up to date:

- **`README.md`**: Project overview, prerequisites, local setup steps (must work for a non-engineer), environment variables reference, how to run tests, how to deploy.
- **`docs/architecture.md`**: High-level diagram of services, data flow, and external integrations.
- **`docs/api.md`**: All API endpoints documented with request/response examples.
- Every exported function must have a JSDoc comment explaining what it does, its parameters, and its return value.

---

## 12. Claude Behaviour Rules

1. **Ask before assuming.** If requirements are ambiguous, ask one clarifying question before writing any code.
2. **Plan before coding.** For any task touching more than one file, output a short bullet-point plan and wait for confirmation before proceeding.
3. **No gold-plating.** Only build what is asked. Do not add unrequested features, even if they seem useful.
4. **Explain decisions.** When making a technical choice (library, pattern, schema design), briefly state why.
5. **Flag risks.** If implementing a feature as described would introduce a security or reliability issue, say so before coding it, and propose a safer approach.
6. **Each task is a coherent change.** Do not produce thousand-line diffs. Break large tasks into reviewable steps.
7. **Test coverage is not optional.** Every new function or component must come with tests. Do not ask if tests are needed.
8. **Respect the dependency policy.** Do not add a new npm package without explicitly stating the justification in plain English (per Section 2 — Third-Party Libraries Policy).
9. **Security by default.** Apply the controls in Section 6 without being asked every time.

---

## 13. Deployment & Environments

- Never deploy manually. Every change reaches production through GitHub Actions only.
- Set up two environments: `local` (development and testing) and `production` (live users). Add staging only if explicitly requested.
- GitHub Actions must run on every PR: lint, type check, tests, security scans (Semgrep, TruffleHog, SCA). All must pass before merge.
- GitHub Actions deploys to production automatically on every merge to `main`.
- All secrets stored in GitHub Actions secrets and the hosting platform's secret manager — never in code.
- Local environment must never point at the production database or any live service.

---

*Authored by: Santiago Kantorowicz*
