# Security Lens

The shared vocabulary and diagnostic lenses for reasoning about **systemic** security weaknesses — design-level issues that span the system, not isolated bugs. Single-sourced here so the two security skills never drift:

- [`review-security`](./review-security/SKILL.md) — autonomous, analysis-only: finds the one dominant weakness and emits a structured finding.
- [`improve-codebase-security`](./improve-codebase-security/SKILL.md) — interactive: presents candidate hardenings, then grills the chosen one toward a secure-by-default design.

## Stance: defensive, not offensive

You read code, reason about design, and recommend defensive direction. You do **not** attempt exploitation and you **never** produce working exploit code or step-by-step attack instructions. Describe the *class* of weakness and how it manifests — the design gap — not how to weaponize it. Framing is "where the design will hurt later," not a pentest report.

## What counts as systemic

A weakness is in scope when the fix would require a **new chokepoint, a refactored trust boundary, or a policy applied across many call sites**. Explicitly out of scope:

- Individual CVEs in dependencies (Dependabot/Snyk territory).
- One-off injection sites when the codebase clearly already parameterizes elsewhere.
- Localized "use bcrypt instead of X" when crypto choice is the only issue and it's confined.
- Compliance checklist items (SOC 2, GDPR specifics) unless they reveal a design weakness.
- Pure infra/network config (firewall, cloud IAM) unless tightly coupled to code.

## Trust boundaries — name them first

Before applying any lens, list the system's trust boundaries explicitly. Most systemic weaknesses live *at* a boundary:

- untrusted ↔ trusted
- tenant ↔ tenant
- user ↔ admin
- client ↔ server
- external ↔ internal
- LLM output ↔ executable surface

## Saltzer & Schroeder — design principles

Evaluate the *shape* of the system, not just specific lines:

- **Economy of mechanism** — is security-critical code small and reviewable, or sprawled? Bespoke crypto, hand-rolled auth, ad hoc session logic are red flags.
- **Fail-safe defaults** — on error or when unconfigured, does it deny or allow by default? Middleware fall-throughs, default route handlers, RLS-off-by-default tables.
- **Complete mediation** — is *every* access to a sensitive resource checked, or only most? Missing chokepoints; authorization decentralized across many handlers.
- **Open design** — does security depend on the design being secret? (predictable IDs treated as secrets, hidden routes assumed safe).
- **Separation of privilege** — are powerful actions guarded by a single check or multiple independent conditions?
- **Least privilege** — what does each component get to do that it doesn't need? Service roles, API keys, DB users, agent tools.
- **Least common mechanism** — do unrelated tenants/users share state, caches, or singletons in a way that leaks across them?
- **Psychological acceptability** — is the secure path the easy path for engineers? If the safe API is harder than the unsafe one, it'll be skipped.

## STRIDE — per trust boundary

- **S**poofing — can a request pretend to be someone it isn't? (auth, session integrity)
- **T**ampering — can data in transit/storage/client-controlled stores be modified? (signing, integrity, client-side trust)
- **R**epudiation — can an action happen without an audit trail?
- **I**nformation disclosure — can data leak via errors, logs, side channels, debug surfaces, response shape?
- **D**enial of service — are there unbounded operations exposed to untrusted callers?
- **E**levation of privilege — can a low-privilege actor reach high-privilege functionality?

## OWASP Top 10 (2021) — pattern library, not checklist

A01 Broken Access Control (IDOR, missing authz, RLS gaps) · A02 Cryptographic Failures · A03 Injection (SQL, command, template, header, **prompt injection in LLM/agent surfaces**) · A04 Insecure Design (missing rate limits, missing tenancy isolation) · A05 Security Misconfiguration (verbose errors, debug routes, permissive CORS) · A06 Vulnerable Components (only if the *usage pattern* is the issue) · A07 Identification/Authentication Failures · A08 Software/Data Integrity Failures (unsigned updates, untrusted deserialization, CI/CD trust) · A09 Logging/Monitoring Failures · A10 SSRF.

## Modern surfaces (if the system uses LLMs/agents)

Prompt injection through user content reaching system/tool prompts · tool-use blast radius · text-to-SQL/code escape · data exfiltration via model output (markdown image/link rendering) · confused deputy (agent acting on its own credentials when it should act on the user's).

## Ranking a finding

When multiple candidates survive, weight by **systemic reach × exploitability × sensitivity of what's at risk**. On a tie, prefer the fix that's a chokepoint or policy change over one that touches every site. Describe the weakness defensively — the design gap, not an exploit recipe.
