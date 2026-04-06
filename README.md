# AWACS — AI Operations with Enforcement

> An AI agent that cannot break its own rules.

**[Live pipeline status →](https://awacs.ai/status)**

---

## The Problem

Every AI agent has the same flaw: it operates on faith.

You write rules. You hope the agent follows them. You find out it didn't when
something goes wrong — if you find out at all.

AWACS is built on a different premise: **the rules are enforced at the tool level,
not the instruction level.** The agent doesn't choose to follow the write chain.
It cannot proceed without following it.

---

## What AWACS Is

A command capture and knowledge pipeline for AI-operated infrastructure, built on
three principles:

**1. Execution over assumption**
Nothing enters the knowledge base unless it was actually run and the output was
observed. No guesses. No docs copy-pasted in. No AI summaries. Executed truth only.

**2. The write chain is enforced, not suggested**
Seven steps from command execution to Class A knowledge. Each step is enforced
by a hook that blocks the tool call if the previous step is incomplete.
The pre-flight check verifies the enforcement infrastructure is active before
any work begins.

**3. Staleness is a first-class concern**
Knowledge has a trust state. Entries progress from provisional → confirmed.
They degrade to suspect, stale, or quarantined. Drift is tracked separately
from staleness. The system knows the difference.

---

## The Write Chain

```
1. Command executes       →  command log (tamper-evident, sha256-hashed)
2. Candidate prepared     →  structured reflection: intent, expected, actual, match
3. Librarian gates        →  5 questions, all must pass, reject is permanent
4. Admitted               →  knowledge-base/class-a/<domain>/<subdomain>/
5. Domain index updated   →  knowledge-base/indexes/
6. Master TOC updated     →  knowledge-base/indexes/master-toc.md
7. Cycle checkpoint       →  captures/cycle-checkpoint-<timestamp>.md
```

Nothing reaches step 4 without steps 1–3 complete and verifiable.
The enforcement hooks block the write if the chain is incomplete.

---

## The Exit 127 Problem

If the hook runtime (Python) is not found, hooks exit with code 127.
Claude Code treats exit 127 as non-blocking — the tool call proceeds.

**This means all enforcement is silently bypassed if Python is not on PATH.**

This is not a theoretical risk. It's a production footgun.

AWACS catches it with a pre-flight check that verifies enforcement is active
before any session work begins. If any of the 28+ checks fail, everything stops.

[Read the write chain spec →](docs/write-chain.md)

---

## Trust Tiers

| Tier | Source | How It Gets There |
|:--|:--|:--|
| **Class A** | Your executed commands | Full write chain only |
| **Class B** | Official vendor docs, SDKs | Source domain rules |
| **Class C** | Community, blogs, forums | Everything else |

Class C never promotes to Class A. Class B never promotes to Class A.
Only execution through the full write chain creates Class A.

[Read the trust tier rules →](docs/trust-tier-rules.md)

---

## What a Class A Entry Looks Like

[See example →](examples/kb-entry-example.md)

---

## Live Pipeline Status

The AWACS pipeline runs on a cron trigger. It updates its own status page.

**[awacs.ai/status](https://awacs.ai/status)** — live feed: Class A count,
domain breakdown, last enforcement event, last admitted entry.

This page is not manually updated. The pipeline writes it.

---

## Current Knowledge Base

49 Class A entries across 7 domains:

| Domain | Entries |
|:--|:--|
| anthropic-claude | 19 |
| tooling | 11 |
| azure-general | 10 |
| github-patterns | 6 |
| atlassian | 1 |
| azure | 1 |
| rubrik | 1 |

80 admits. 12 rejects. 0 pending.

---

## What This Opens Up

These are production outcomes from running AWACS on live infrastructure:

**Alert fatigue → actionable intelligence**
100+ daily backup alerts processed, transient failures separated from persistent ones, confirmed incidents routed to ServiceNow automatically. Zero manual triage. One engineer's intuition, built over years, replaced by a system that doesn't take sick days.
[API Voyager case study →](https://awacs.ai/case-studies/api-voyager.html)

**AI that can't repeat its own mistakes**
An AI agent modified `sshd_config` on an Arc-connected node — restricting it to IPv4 while Arc SSH uses IPv6 — and caused a multi-hour lockout. AWACS captured that as a Class A anti-pattern entry. The next session knew about it before touching the node.
[Doctrine enforcement case study →](https://awacs.ai/case-studies/doctrine-enforcement.html)

**Vendor docs that are wrong, caught and corrected**
The Azure Local HCI Orchestrator silently overwrites WDAC policies deployed via `CiTool.exe` within 90 minutes. The vendor's documentation says to use `CiTool.exe`. That's wrong. AWACS captured the correct path, validated it, stored it permanently, and flags it for revalidation if the dependency changes.
[WDAC deployment case study →](https://awacs.ai/case-studies/wdac-ai-deployment.html)

**Audit-ready AI operations**
Every AI action is logged. Every knowledge base write has a traceable decision chain: command log → candidate → gate decision → admission. Compliance auditors can follow the entire chain for any entry.

---

## Executive Summary

[Read the full executive summary →](EXECUTIVE-SUMMARY.md)

---

## Want to Build This for Your Stack?

This repo contains the methodology. The implementation — the hooks, the pipeline,
the aiOS modules that connect to Azure, Rubrik, and Atlassian — is the engagement.

Two ways to connect:

**Follow for the methodology:**
[linkedin.com/in/dustin-winkler-nc](https://www.linkedin.com/in/dustin-winkler-nc/)

**Build it for your environment:**
[Book a strategy call](https://calendly.com/dustin-dustinwinkler/30min) · [dustin@awacs.ai](mailto:dustin@awacs.ai?subject=AWACS%20-%20Let%27s%20build%20this)

---

*AWACS is the methodology. The implementation stays private.
The rules are public. The enforcement is the product.*
