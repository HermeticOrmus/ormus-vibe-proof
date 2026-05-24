# ormus-vibe-proof

> Security hardening for vibe-coded full-stack apps. Three parallel audit agents, seven categories, prioritized fix queue.

A Claude Code skill that systematically audits and hardens security vulnerabilities in vibe-coded full-stack applications. Three audit agents run in parallel across frontend, backend, and config layers; findings get deduplicated, prioritized by severity, and fixed in order.

## Why

Vibe coding gets you to working software fast. It also tends to leave behind:

- SQL injection in `ORDER BY` clauses (everyone remembers `WHERE`, nobody remembers `ORDER BY`)
- Hardcoded "temporary" passwords that became permanent backdoors
- API tokens passed in URL query params instead of Authorization headers
- `.env` files committed to git
- Missing security headers (HSTS, X-Frame-Options, CSP)
- Error responses that leak internals (`err.message` straight to the client)
- Missing input validation on enum fields (gender, status, role)
- Dead code paths still reachable through old routes

`/vibe-proof` runs the same audit-then-fix loop a competent security engineer would, in parallel, with prioritized output. Designed for the moment between "MVP shipped" and "first real customer."

## Install

```bash
git clone https://github.com/HermeticOrmus/ormus-vibe-proof ~/.claude/skills/vibe-proof
```

Or as a Claude Code plugin:

```bash
claude plugin marketplace add HermeticOrmus/ormus-vibe-proof
```

Restart Claude Code so the skill registry picks it up.

## Usage

```
/vibe-proof
```

Or trigger via natural language: "security audit this project", "harden the security", "vibe-code proof this", "make this secure".

## What gets audited

| # | Category | Sample checks |
|---|---|---|
| 1 | **Injection vectors** | SQL parameterization, sort-column allowlists, no `eval()`, bounds on numeric params |
| 2 | **PII & secret exposure** | Hardcoded creds, tokens in URLs, `.env` in git, public env vars exposing private values |
| 3 | **Missing security headers** | HSTS, X-Frame-Options, X-Content-Type-Options, CSP, Referrer-Policy |
| 4 | **Error leakage** | Stack traces in production, `err.message` to client, sensitive data in logs |
| 5 | **Input validation gaps** | Zod-validated bodies, enum allowlists, magic-byte file checks, MIME-derived extensions |
| 6 | **Dead code & attack surface** | Unused routes, GET-aliased-POST handlers, disabled features still reachable |
| 7 | **Credential hygiene** | Cookie flags, session secret length, trust-proxy config, webhook signature verification |

## How it works

```
Phase 1: Parallel audit (read-only)
  → Agent 1: Frontend
  → Agent 2: Backend / API
  → Agent 3: Config & credentials

Phase 2: Synthesize & prioritize
  → Dedupe overlapping findings
  → Severity-rank: CRITICAL → HIGH → MEDIUM → LOW

Phase 3: Systematic fix execution
  → Fix in priority order
  → npm run build between categories
  → Verify no regressions

Phase 4: Credential remediation
  → Untrack any leaked .env files
  → Rotate exposed secrets

Phase 5: Verify & deploy
Phase 6: Post-deploy connection verification
```

The skill includes ~10 ready-to-paste fix patterns for the most common findings: backdoor password removal, MIME-based file extensions, enum allowlist validation, SQL injection allowlist pattern, API tokens in headers, security header config (Next.js + Express), error response masking, GET-as-POST removal.

## Subagent requirement

Phase 1 launches three parallel audit agents. By default the skill uses the `general-purpose` subagent type, which is always available in Claude Code. The audit prompts are detailed enough to drive any capable agent.

If you have specialized security agents installed via plugins, swap them in by editing the agent invocation in your local copy of `SKILL.md`.

## Pairs with

- **[ormus-handoff](https://github.com/HermeticOrmus/ormus-handoff)** — capture session state. Run after a long /vibe-proof session before clearing context.
- **[ormus-pickup](https://github.com/HermeticOrmus/ormus-pickup)** — restore context next session.
- **[ormus-absorb](https://github.com/HermeticOrmus/ormus-absorb)** — distill what an audit taught you about a project's threat surface into persistent memory.
- **[ormus-explore](https://github.com/HermeticOrmus/ormus-explore)** — token-efficient AST-based code search. Useful during audit for surfacing suspect patterns across a codebase.
- **[ormus-meta-prompting](https://github.com/HermeticOrmus/ormus-meta-prompting)** — categorical foundations for AI prompt engineering.

Together they form the **ormus session lifecycle** — composable Claude Code skills for serious cross-day, cross-machine work.

## License

MIT. See [LICENSE](LICENSE).

## Origin

Distilled from real hardening sessions on production full-stack apps. Across multiple platforms (React + Express + payment integrations, Next.js + Supabase + CRM): 85+ issues surfaced — SQL injection, hardcoded backdoor passwords, secrets in URL params, `.env` files in git, missing security headers, error leakage. The seven-category checklist and the priority order both emerged from doing this work the manual way enough times that the patterns codified themselves.

---

## Part of the Libre Open-Source Stack for Claude Code

This repository is part of a growing family of open-source toolkits for Claude Code.

### Libre suite — comprehensive plugin bundles

- [LibreUIUX-Claude-Code](https://github.com/HermeticOrmus/LibreUIUX-Claude-Code) — UI/UX development (152 agents, 70 plugins, 76 commands, 74 skills)
- [LibreArch-Claude-Code](https://github.com/HermeticOrmus/LibreArch-Claude-Code) — Software architecture and system design
- [LibreCopy-Claude-Code](https://github.com/HermeticOrmus/LibreCopy-Claude-Code) — Technical writing and documentation engineering
- [LibreDevOps-Claude-Code](https://github.com/HermeticOrmus/LibreDevOps-Claude-Code) — DevOps engineering and infrastructure automation
- [LibreEmbed-Claude-Code](https://github.com/HermeticOrmus/LibreEmbed-Claude-Code) — Embedded systems, firmware, and IoT development
- [LibreFinTech-Claude-Code](https://github.com/HermeticOrmus/LibreFinTech-Claude-Code) — Financial technology development
- [LibreGEO-Claude-Code](https://github.com/HermeticOrmus/LibreGEO-Claude-Code) — AI-search optimization (ChatGPT, Perplexity, Gemini, Google AI Overviews)
- [LibreGameDev-Claude-Code](https://github.com/HermeticOrmus/LibreGameDev-Claude-Code) — Game development across Godot, Unity, Unreal
- [LibreMLOps-Claude-Code](https://github.com/HermeticOrmus/LibreMLOps-Claude-Code) — ML engineering and AI operations
- [LibreMobileDev-Claude-Code](https://github.com/HermeticOrmus/LibreMobileDev-Claude-Code) — Mobile app development (Flutter, React Native, native iOS, native Android)
- [LibreSecOps-Claude-Code](https://github.com/HermeticOrmus/LibreSecOps-Claude-Code) — Security operations

### Skills mini-repos — single CLAUDE.md drop-ins

- [vibe-engineer-skills](https://github.com/HermeticOrmus/vibe-engineer-skills) — Direct AI codegen well (hypothesis → scope → validate → reject working-but-wrong)
- [markdown-discipline-skills](https://github.com/HermeticOrmus/markdown-discipline-skills) — Strip AI-slop from markdown (no em dashes, no marketing fluff)
- [shell-safety-skills](https://github.com/HermeticOrmus/shell-safety-skills) — `set -euo pipefail` discipline + 15 failure-mode examples
- [commit-standard-skills](https://github.com/HermeticOrmus/commit-standard-skills) — Ormus Commit Standard v1.0 + commit-msg hook + commitlint
- [unwoke-skills](https://github.com/HermeticOrmus/unwoke-skills) — Strip AI theater (ten sins to eliminate, symmetric engagement)
- [python-conventions-skills](https://github.com/HermeticOrmus/python-conventions-skills) — Modern Python 3.11+ (types, pathlib, async, ruff, mypy, uv)
- [typescript-conventions-skills](https://github.com/HermeticOrmus/typescript-conventions-skills) — TypeScript strict mode, discriminated unions, Result types
- [hermetic-laws-skills](https://github.com/HermeticOrmus/hermetic-laws-skills) — Seven Hermetic Principles applied to engineering
- [riper-workflow-skills](https://github.com/HermeticOrmus/riper-workflow-skills) — Research / Innovate / Plan / Execute / Review systematic dev
- [six-day-cycle-skills](https://github.com/HermeticOrmus/six-day-cycle-skills) — Sustainable shipping cadence with mandatory rest
- [token-optimization-skills](https://github.com/HermeticOrmus/token-optimization-skills) — Claude Code token + context optimization
- [osint-skills](https://github.com/HermeticOrmus/osint-skills) — OSINT research methodology (multi-wave investigative spiral)
- [calcinate-skills](https://github.com/HermeticOrmus/calcinate-skills) — Stage 1 of the Magnum Opus (burn project bloat)
- [claude-md-overhaul-skills](https://github.com/HermeticOrmus/claude-md-overhaul-skills) — Audit CLAUDE.md and MEMORY.md against caps
- [session-handoff-skills](https://github.com/HermeticOrmus/session-handoff-skills) — Session handoff + pickup discipline
- [naming-skills](https://github.com/HermeticOrmus/naming-skills) — Product naming methodology (mine the brand's vocabulary)
- [magnum-opus-skills](https://github.com/HermeticOrmus/magnum-opus-skills) — Seven-stage alchemy applied to project transformation

### Template source

- [andrej-karpathy-skills](https://github.com/HermeticOrmus/andrej-karpathy-skills) — the canonical single-file CLAUDE.md pattern (fork of jiayuan_jy's original)

Star the family, not just one — that's how the suite stays coherent.
