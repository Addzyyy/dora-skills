---
name: dependency-management
description: Apply when running npm/pip/cargo install, adding packages, upgrading libraries, resolving lockfile conflicts, or auditing for vulnerabilities. Lock exact versions, commit lockfiles, and scan before merging
---

# Dependency Management

## Overview

Every dependency is code you did not write but must maintain. Outdated dependencies accumulate known vulnerabilities, fall out of support, and create upgrade cliffs. Managed dependency hygiene — lockfiles, regular updates, and vulnerability scanning — keeps the supply chain from becoming a production risk.

## DORA Impact

| Metric | Effect |
|--------|--------|
| Lead Time for Changes | Current dependencies reduce upgrade friction; fewer breaking changes to resolve before a feature ships |
| Change Failure Rate | Patched dependencies eliminate known vulnerabilities and compatibility issues before they cause incidents |

## When to Use

- Adding a new library to the project
- Updating an existing dependency (security patch, minor, or major)
- Auditing the dependency tree for vulnerabilities or abandoned packages
- Onboarding to a codebase to understand its dependency health
- A CI scan flags a known CVE in a transitive dependency

## When NOT to Use

- A truly isolated script with no deployment surface and no dependencies worth tracking
- A vendored dependency that is intentionally pinned and managed separately — do not mix strategies mid-project

## Core Pattern

**Before — unmanaged dependency state:**

```
requirements.txt (or package.json without lockfile):
  express: ^4.0.0        ← resolves to different versions on each install
  lodash: latest         ← a known prototype-pollution CVE exists in the installed version
  left-pad: 0.0.3        ← abandoned, no updates in 4 years

No lockfile committed.
No vulnerability scan in CI.
npm install on a new machine resolves to a different version set than production.
→ "Works on my machine" failures; CVE exploited in production.
```

**After — managed dependency hygiene:**

```
Lockfile committed:
  express: 4.18.2         ← exact version pinned; every machine installs identically
  lodash: 4.17.21         ← patched version; CVE resolved

CI pipeline:
  Step 1: install from lockfile (fail if lockfile is out of sync)
  Step 2: run vulnerability scan → fail build on HIGH or CRITICAL CVEs
  Step 3: run tests

Dependency update cadence:
  Weekly: automated PRs for patch and minor updates (auto-merge if CI passes)
  Monthly: review major version updates manually
  Immediate: security patches on HIGH/CRITICAL CVEs
```

**Evaluating a new dependency:**

```
Before adding, answer:
  [ ] Does this library do one thing I cannot reasonably write myself in < 2 hours?
  [ ] Is it actively maintained? (last commit, open issues, release cadence)
  [ ] How large is it? Does it pull in a large transitive dependency tree?
  [ ] Does it have known CVEs?
  [ ] Is there a lighter-weight or already-approved alternative in the project?
  [ ] What is the license? Is it compatible with the project's license?
```

**Lockfile discipline:**

```
Rule: Always commit the lockfile.
Rule: Never edit the lockfile by hand.
Rule: Regenerate the lockfile using the package manager's update command.
Rule: CI fails if the committed lockfile does not match the manifest.
```

**Vulnerability scanning:**

```
Integrate into CI:
  - Run the package manager's audit command (e.g., npm audit, pip-audit, bundler-audit)
  - Fail on HIGH and CRITICAL severity
  - Log MEDIUM for review; do not fail the build (avoid alert fatigue)
  - Re-run on a schedule (nightly) to catch newly disclosed CVEs in pinned versions
```

## Quick Reference

| Rule | Guidance |
|------|----------|
| Lockfile | Always commit; CI fails if out of sync with the manifest |
| Update cadence | Patches/minor weekly (automated); major monthly (manual review) |
| CVE response | HIGH/CRITICAL patched within one sprint; blocks merge if unresolved |
| New dependency bar | Solves a real problem, actively maintained, minimal transitive footprint |
| Vulnerability scan | Runs in CI on every PR and on a nightly schedule |

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Not committing the lockfile | Different environments resolve different versions; flaky builds and hidden CVEs | Commit the lockfile; add it to the repository from day one |
| Pinning to `latest` | Breaks unexpectedly when a major version ships | Pin to a specific version or range in the manifest; let the lockfile fix the exact version |
| Ignoring vulnerability scan output | Known CVEs accumulate until they become incidents | Treat HIGH/CRITICAL as build failures; assign ownership of MEDIUM within a sprint |
| Adding dependencies for trivial utilities | Expands the attack surface and transitive tree for no meaningful gain | Write small utilities inline; reserve dependencies for non-trivial, well-maintained libraries |
| Deferring major version upgrades indefinitely | Creates an upgrade cliff: one day all majors must land at once | Review major updates monthly; upgrade one major at a time, with tests, on a quiet week |

## Related Skills

- **type-safety-and-linting** — dependency updates can introduce type errors; re-run strict checks after every upgrade
- **configuration-as-code** — dependency versions are configuration that belongs in version-controlled files
- **contract-testing** — dependency updates can break consumer contracts; verify contracts after upgrades
