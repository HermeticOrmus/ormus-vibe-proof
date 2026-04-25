---
name: vibe-proof
description: |
  Security-focused hardening for vibe-coded full-stack apps. Runs parallel
  audits across frontend, backend, and config layers, then fixes issues
  systematically by severity. Covers injection, PII exposure, missing
  headers, error leakage, dead code, and credential hygiene.
triggers:
  - /vibe-proof
  - security audit this project
  - harden the security
  - vibe-code proof this
  - make this secure
---

# Vibe-Proof: Security Hardening for Full-Stack Apps

**Purpose:** Systematically audit and fix security vulnerabilities in vibe-coded full-stack applications through parallel multi-agent analysis and guided remediation.

## When to Use

- After vibe-coding an MVP with API routes, databases, or payment integrations
- Before first real deployment or first real customer
- When you suspect "it works but is it safe?"
- Any Express/React/Next.js/Nuxt/full-stack app with a backend

## The Seven Security Checks

### 1. Injection Vectors
- [ ] No user input in SQL/query strings without parameterization
- [ ] Sort columns, filter fields use allowlist validation
- [ ] No `eval()`, `new Function()`, or template literal injection
- [ ] URL params parsed with bounds checking (parseInt with min/max)
- [ ] Enum fields (gender, status, role) validated against const allowlists

### 2. PII & Secret Exposure
- [ ] No hardcoded addresses, phone numbers, names in source
- [ ] No hardcoded passwords or "backdoor" auth strings
- [ ] API tokens in headers (Authorization), never in URL params
- [ ] Admin endpoint secrets use `Authorization: Bearer` header, not query params
- [ ] No `.env` files tracked in git (check `git ls-files | grep env`)
- [ ] No secrets in client-side code or `VITE_*` / `NEXT_PUBLIC_*` vars that shouldn't be public
- [ ] `.env.example` documents all required variables (sync secrets, third-party API keys, etc.)
- [ ] No `localhost` URLs in production allowlists (ALLOWED_ORIGINS, CSP, etc.)

### 3. Missing Security Headers
- [ ] `X-Content-Type-Options: nosniff`
- [ ] `X-Frame-Options: DENY` (or SAMEORIGIN if iframes needed)
- [ ] `X-XSS-Protection: 0` (modern best practice — disables buggy browser filter)
- [ ] `Referrer-Policy: strict-origin-when-cross-origin`
- [ ] `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`
- [ ] `X-DNS-Prefetch-Control: off` (privacy — prevents browser DNS leaks)
- [ ] Body size limits on `express.json()` and `express.urlencoded()` (Express)
- [ ] CSP `img-src` restricted to specific CDN domains (not `https:` wildcard)
- [ ] CSP `script-src` without `unsafe-eval` (remove if WebGL/shaders were deleted)

### 4. Error Leakage
- [ ] Production error responses don't expose stack traces
- [ ] 500 errors return generic message, not `error.message`
- [ ] No `console.log` of sensitive data (tokens, passwords, PII)
- [ ] Structured logger used instead of `console.*` in production code
- [ ] Catch blocks return masked errors: `"Internal server error"` not `err.message`

### 5. Input Validation Gaps
- [ ] All POST/PUT endpoints validate body with Zod or equivalent
- [ ] Query params have type coercion and bounds (limit, offset, id)
- [ ] Integer params checked against MAX_INT (2147483647)
- [ ] Enum params validated against `const ALLOWED_X = [...] as const` allowlists
- [ ] File uploads check size AND validate magic bytes (not just MIME header)
- [ ] File extensions derived from validated MIME type, not user-supplied filename
- [ ] Token/secret params validated for format (min length, charset) before DB lookup
- [ ] Text inputs sanitized (strip HTML tags, dangerous chars) before storage

### 6. Dead Code & Attack Surface
- [ ] Unused routes/endpoints removed
- [ ] Unused components deleted (not commented out)
- [ ] Disabled features removed entirely (not just `if(false)`)
- [ ] Test/debug endpoints not in production
- [ ] Unused npm packages removed
- [ ] No GET handler aliasing POST on write endpoints (`export { POST as GET }`)
- [ ] No conflicting static + dynamic files (e.g., `robots.txt` + `robots.ts`)
- [ ] Unused client utility functions removed (dead `createBrowserClient`, etc.)
- [ ] Embedded video/iframe sources use privacy-enhanced mode where available

### 7. Credential Hygiene
- [ ] Session secrets are 32+ characters
- [ ] Cookies: `httpOnly`, `secure` (production), `sameSite: 'lax'`
- [ ] Trust proxy configured when behind reverse proxy (Vercel, nginx)
- [ ] Webhook endpoints verify signatures (Stripe, GitHub, etc.)
- [ ] Rate limiting on auth, checkout, newsletter, AND admin/sync endpoints
- [ ] Rate limiting strategy appropriate for platform (in-memory is defense-in-depth on serverless; use a persistent store for real protection)

## Execution Process

### Phase 1: Parallel Audit (Read-Only)

Launch three audit agents in parallel, each scanning a different layer. Use the `general-purpose` subagent type by default (always available). If your Claude Code installation has specialized security agents, swap them in — but the audit prompts below are detailed enough to drive any capable agent.

**Agent 1: Frontend Audit**

```
Audit the frontend code for security issues:
- XSS vectors (dangerouslySetInnerHTML, unescaped user input)
- Sensitive data in client-side code
- Tracking pixels with undefined variables
- Console.log statements leaking data
- Dead/unused components
- API keys or tokens in VITE_*/NEXT_PUBLIC_* env vars that shouldn't be public
- Embedded videos/iframes not using privacy-enhanced mode
- sessionStorage/localStorage holding PII unnecessarily
Report each issue with file:line, severity, and fix suggestion.
```

**Agent 2: Backend / API Audit**

```
Audit the backend code for security issues:
- SQL injection (user input in query strings, unvalidated sort/filter)
- Missing input validation on POST/PUT endpoints
- Hardcoded PII (addresses, phone numbers, names)
- Hardcoded passwords or backdoor auth strings
- API tokens or secrets in URL params instead of Authorization header
- GET handlers that alias POST on write endpoints
- console.log/error statements (should use structured logger)
- Error handlers leaking internal details (returning err.message to client)
- Missing rate limiting on sensitive endpoints (including admin/sync)
- Missing enum/allowlist validation on fields like gender, status, role
- File extension derived from user filename instead of validated MIME type
- Duplicate utility code that should be extracted to shared modules
Report each issue with file:line, severity, and fix suggestion.
```

**Agent 3: Config & Credential Audit**

```
Audit configuration and credentials:
- .env files tracked in git (git ls-files | grep env)
- .env.example missing required variables
- Security headers present/missing (check next.config headers or Express middleware)
- X-XSS-Protection should be "0" (not "1; mode=block")
- HSTS header with adequate max-age (63072000+) and preload
- CSP img-src using wildcard https: instead of specific domains
- CSP script-src with unnecessary unsafe-eval
- localhost URLs in production allowlists (ALLOWED_ORIGINS, CSP connect-src)
- Body size limits configured (Express)
- Session configuration (secret length, cookie flags)
- Trust proxy setting
- Conflicting static + dynamic files (robots.txt vs robots.ts)
- Dead code files (unused components, disabled features)
- Unused npm dependencies (especially heavy ones)
Report each issue with file:line, severity, and fix suggestion.
```

### Phase 2: Synthesize & Prioritize

Combine all findings into a single prioritized list:

| Priority | Category | Fix Order |
|----------|----------|-----------|
| CRITICAL | Backdoor passwords, injection, credential leaks, secrets in URLs | Fix first |
| HIGH | PII exposure, missing validation, error leakage, missing HSTS, GET-as-POST | Fix second |
| MEDIUM | Missing rate limits, enum validation, dead code, CSP tightening | Fix third |
| LOW | Unused packages, console.log, config optimization | Fix last |

**Deduplication**: Agents will find overlapping issues. Merge duplicates and keep the most detailed description.

### Phase 3: Systematic Fix Execution

Fix in priority order. After each fix category:
1. Run `npm run build` (or project equivalent)
2. Verify no regressions

**Common Fix Patterns:**

#### Backdoor Password Removal
```typescript
// BEFORE: Hardcoded backdoor (CRITICAL)
const password = searchParams.get("password");
if (password !== "myapp2024") return unauthorized();

// AFTER: Environment variable via Authorization header
const authHeader = request.headers.get("authorization");
const secret = authHeader?.replace(/^Bearer\s+/i, "");
if (!process.env.SYNC_SECRET || !secret || secret !== process.env.SYNC_SECRET) {
  return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
}
```

#### Enum Allowlist Validation
```typescript
const ALLOWED_GENDERS = ["male", "female", "other"] as const;
const ALLOWED_LANGUAGES = ["es", "en"] as const;

if (!ALLOWED_GENDERS.includes(data.gender)) {
  return NextResponse.json({ error: "Invalid gender value" }, { status: 400 });
}
```

#### MIME-Based File Extension
```typescript
// BEFORE: Trust user filename (attackable)
const ext = file.name.split('.').pop();

// AFTER: Derive from validated MIME type
const MIME_TO_EXT: Record<string, string> = {
  "application/pdf": "pdf",
  "image/jpeg": "jpg",
  "image/png": "png",
  "image/webp": "webp",
};
const ext = MIME_TO_EXT[file.type] || "bin";
```

#### DRY: Extract Shared Service Clients
```typescript
// BEFORE: createClient() duplicated in 3+ route files

// AFTER: Shared module
import { createClient, SupabaseClient } from "@supabase/supabase-js";

export function createServiceClient(): SupabaseClient | null {
  const url = process.env.SUPABASE_URL;
  const key = process.env.SUPABASE_SERVICE_KEY;
  if (!url || !key) return null;
  return createClient(url, key);
}
```

#### SQL Injection (Allowlist Pattern)
```typescript
const ALLOWED_SORT_COLUMNS: Record<string, string> = {
  'created': 'created_at',
  'rating': 'average_rating',
  'name': 'name',
  'price': 'price::numeric',
};

if (filters?.sortBy && ALLOWED_SORT_COLUMNS[filters.sortBy]) {
  const sortColumn = ALLOWED_SORT_COLUMNS[filters.sortBy];
  query += ` ORDER BY ${sortColumn}`;
}
```

#### API Token in Header (Not URL)
```typescript
// BEFORE: Token in URL (visible in logs, browser history)
const url = `${API_URL}?access_token=${TOKEN}`;

// AFTER: Token in Authorization header
fetch(API_URL, {
  headers: { 'Authorization': `Bearer ${TOKEN}` }
});
```

#### Security Headers — Next.js (next.config.ts)
```typescript
{ key: "X-Content-Type-Options", value: "nosniff" },
{ key: "X-Frame-Options", value: "DENY" },
{ key: "X-XSS-Protection", value: "0" },  // Modern: disable buggy filter
{ key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
{ key: "Strict-Transport-Security", value: "max-age=63072000; includeSubDomains; preload" },
{ key: "X-DNS-Prefetch-Control", value: "off" },
```

#### Security Headers — Express (middleware)
```typescript
app.use((_req, res, next) => {
  res.setHeader('X-Content-Type-Options', 'nosniff');
  res.setHeader('X-Frame-Options', 'DENY');
  res.setHeader('X-XSS-Protection', '0');
  res.setHeader('Referrer-Policy', 'strict-origin-when-cross-origin');
  res.setHeader('Strict-Transport-Security', 'max-age=63072000; includeSubDomains; preload');
  next();
});
```

#### Error Response Masking
```typescript
// Express
app.use((err, _req, res, _next) => {
  const status = err.status || 500;
  res.status(status).json({
    error: status >= 500 ? 'Internal Server Error' : err.message
  });
});

// Next.js route handler
} catch (err) {
  console.error("Route error:", err);  // Log internally
  return NextResponse.json(
    { error: "Internal server error" },  // Mask externally
    { status: 500 }
  );
}
```

#### Remove GET-as-POST Alias
```typescript
// BEFORE: Exposes write endpoint to GET requests (CSRF, caching, logging)
export async function GET(request: NextRequest) {
  return POST(request);
}

// AFTER: Delete the GET export entirely. Only export POST.
```

### Phase 4: Credential Remediation

If `.env` files were tracked in git:

```bash
echo ".env.production" >> .gitignore
git rm --cached .env.production
# Then rotate ALL exposed credentials.
```

**Credentials that MUST be rotated if exposed:**
- Database passwords / service-role keys
- Third-party API keys (payment processor, CRM, email, etc.)
- Session/sync secrets
- Webhook signing secrets

### Phase 5: Verify & Deploy

```bash
npm run build
git commit -m "fix(security): harden platform — [summary of fixes]"
git push
```

### Phase 6: Post-Deploy Connection Verification

After deploy, verify all external services are reachable from production:

1. **Test each service endpoint** with a minimal query
2. **Check for common failures**:
   - `ENOTFOUND` / `NXDOMAIN` → managed service paused or DNS misconfigured
   - `401 Unauthorized` → API key mismatch between local `.env` and production env vars
   - Schema cache errors → service just restored, wait 1-2 min
3. **Clean up any test records** created during verification

## Success Criteria

- [ ] Build passes with zero warnings
- [ ] No user input reaches SQL without parameterization or allowlist
- [ ] No PII or hardcoded passwords in source code
- [ ] No API tokens or secrets in URLs
- [ ] No .env files tracked in git
- [ ] Security headers present on all responses (HSTS, DENY, nosniff)
- [ ] Error responses don't leak internals
- [ ] All POST/PUT endpoints validate input (including enum allowlists)
- [ ] File extensions derived from MIME type, not filename
- [ ] Dead code deleted (not commented)
- [ ] No GET aliases for POST endpoints
- [ ] Exposed credentials rotated
- [ ] All external service connections verified post-deploy

## Lessons Learned

1. **Parallel audit agents save massive time.** Three agents scanning different layers simultaneously catch issues that sequential review misses.

2. **Sort columns are the #1 SQL injection vector in vibe-coded apps.** Everyone parameterizes WHERE clauses but forgets ORDER BY.

3. **`.env` files in git are shockingly common.** Always check `git ls-files | grep -i env` as the very first step.

4. **Hardcoded "temporary" passwords become permanent backdoors.** Search for string comparisons against literals in auth logic: `password !== "something"`, `secret === "hardcoded"`.

5. **GET aliasing POST is a silent CSRF vector.** `export { POST as GET }` or `GET(req) { return POST(req) }` exposes write endpoints to CSRF, browser prefetch, and CDN caching.

6. **`X-XSS-Protection: 1; mode=block` is outdated.** Modern recommendation is `0` — the browser filter itself has been exploited. CSP is the real protection.

7. **Enum validation is easy to forget.** Gender, status, role, language — any field with a finite set of values needs a const allowlist, not just type checking.

8. **File extension from filename is attackable.** A file named `malware.pdf.exe` with `application/pdf` MIME should get `.pdf` extension from MIME, not `.exe` from the name.

9. **Duplicate service-client code = duplicate security gaps.** Extract shared modules so auth logic is fixed in one place.

10. **`localhost` in production origin allowlists is a real finding.** Easy to leave `http://localhost:3000` in `ALLOWED_ORIGINS` during development.

11. **Always test connections after deploy.** A passing build doesn't prove services are reachable. Test each external service with a minimal query post-deploy.

12. **Free-tier managed services pause after inactivity.** DNS records can disappear (NXDOMAIN). After unpause, schema caches take 1-2 min to warm. Production may work before local due to separate connection pools.
