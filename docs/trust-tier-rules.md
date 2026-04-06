# AWACS Trust Tier Rules

> The canonical reference for how AWACS classifies and manages knowledge.
> Nothing enters Class A without passing the full write chain.

---

## The One Rule That Cannot Be Broken

**Class C never promotes to Class A. Class B never promotes to Class A.**

Only execution through the full write chain creates Class A entries:

```
command executed → output logged → candidate prepared → librarian gate → gate decision → Class A write
```

No amount of corroboration, cross-referencing, or community validation substitutes
for running the command and observing the result.

---

## The Three Tiers

| Tier | Name | What It Is | Source |
|:--|:--|:--|:--|
| **Class A** | Validated operational truth | Commands that were run, outputs that were observed, patterns that were confirmed | Your own executed commands only |
| **Class B** | Official vendor sources | Vendor docs, official SDKs, published API references, changelogs | Vendor documentation sites, official GitHub orgs |
| **Class C** | Community / unverified | Forum posts, blog articles, community discussions, practitioner content | Everything else |

Class A is not "things we believe." It is "things we ran and observed."

---

## The Librarian Admission Gate (5 Questions)

Every candidate entry must pass all five before entering Class A:

1. **Was this genuinely validated?** Actually executed — not hypothetical, not copied from docs.
2. **Is it reusable?** Likely to be useful again in a similar task or environment.
3. **Does it have minimum required metadata?** Intent, scope, expected output, actual output.
4. **Is it structured enough to retrieve intelligently?** Not a raw blob.
5. **Is it worth remembering as reusable operational truth?** The master question.

If any answer is no: reject. Not defer, not maybe. Reject.

### What Never Gets Admitted

- Unvalidated ideas or candidate commands never run
- Generic reference information (that's Class B)
- Contextless successes: worked once but no scope, no reason, no expected output
- AI-generated summaries from other tools
- Pure speculation

---

## Class A Entry Schema

Every Class A entry carries required frontmatter:

```yaml
---
id: cmd-YYYYMMDD-HHMM-NNN
timestamp: "2026-03-29T10:30:00Z"
domain: azure | rubrik | anthropic-claude | linux | github-patterns | tooling
sub_discipline: hooks | vm-provisioning | rbac | policy | etc
trust_state: a-provisional | a-confirmed | a-scoped | a-suspect | a-sporadic | a-stale
validated_by: sonnet | opus
output_type: deterministic | state-dependent | variable
risk_class: read-only | mutating | troubleshooting-probe | scoped-fact | anti-path
revalidation_eligible: true | false
validation_count: 1
---
```

---

## Trust State Lifecycle

| State | Meaning |
|:--|:--|
| **A-Provisional** | Worked once. Worth storing. Not yet hardened. |
| **A-Confirmed** | Worked repeatedly. Preferred in matching scope. |
| **A-Scoped** | Confirmed, but only valid for a specific environment or version. |
| **A-Suspect** | Recent mismatch creates doubt. Under review. |
| **A-Sporadic** | Mixed success/failure. Not reliable as default. |
| **A-Stale** | Not recently validated or dependency changed. Queue for retest. |
| **A-Quarantined** | Do not use. History only. Confirmed wrong or dangerous. |

```
New validated result
        │
        ▼
[Librarian Gate] ──REJECT──► discard
        │
      ADMIT
        │
        ▼
  A-Provisional
        │ (revalidated successfully)
        ▼
  A-Confirmed
        │ (failure in matching scope)
        ▼
  A-Suspect ──► A-Stale | A-Sporadic | A-Quarantined
```

---

## Staleness vs Drift — A Critical Distinction

**Staleness** = the knowledge is no longer accurate. Tool changed. API deprecated.
**Drift** = the environment moved. The knowledge is still correct.

These require different responses. Staleness updates the KB entry. Drift writes a drift event and leaves the entry alone.

| Situation | Interpretation |
|:--|:--|
| Command syntax no longer accepted | Stale |
| Flag renamed or deprecated | Stale |
| Command valid but target unavailable | Drift signal |
| Command valid but auth fails unexpectedly | Drift signal |
| Works intermittently after prior consistency | Sporadic, possibly drift |

Re-validate on dependency change — not on a timer.
