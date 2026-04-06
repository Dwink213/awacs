# The AWACS Write Chain

> The write chain is the enforcement mechanism that separates AWACS from a system
> that just hopes the AI behaves correctly. Every step is enforced at runtime.
> Skipping a step is not possible — the system blocks it.

---

## The Seven Steps

```
1. Command executes        →  captures/command-log-YYYY-MM-DD.jsonl
2. Candidate prepared      →  captures/candidates/<entry-id>.md
3. Librarian gates         →  captures/librarian-decisions.jsonl
4. If admitted             →  knowledge-base/class-a/<domain>/<subdomain>/<entry-id>.md
5. Domain index updated    →  knowledge-base/indexes/domain-index-<domain>.md
6. Master TOC updated      →  knowledge-base/indexes/master-toc.md
7. Cycle checkpoint        →  captures/cycle-checkpoint-<timestamp>.md
```

Nothing reaches step 4 without steps 1–3 being complete and verifiable.
The enforcement hooks block the write at the tool level if the chain is incomplete.

---

## Why Each Step Exists

**Step 1 — Command log**
Creates the tamper-evident audit trail. Output is sha256-hashed. If the hash doesn't match, the entry is suspect. The command log is the source of truth for "did this actually run."

**Step 2 — Candidate file**
Forces structured reflection before admission. The analyzer must articulate intent, expected output, actual output, and match quality. This step is what separates "it ran" from "it ran and we understand why."

**Step 3 — Librarian gate**
Five questions. All must pass. A reject is permanent — the entry does not sit in a queue. If it wasn't good enough on first review, it goes to the discard pile, not a maybe pile.

**Step 4 — Class A write**
Only happens after an explicit `admit` decision in the decisions log. The PreToolUse hook checks this before allowing the write. The AI cannot write to Class A by deciding to — it can only do so after the gate has recorded a decision.

**Steps 5–7 — Housekeeping enforcement**
Domain indexes and cycle checkpoints are not optional. The hooks block the *next* Class A write if the index wasn't updated after the last round. You cannot outrun the bookkeeping.

---

## What the Hooks Actually Enforce

| Hook | When It Fires | What It Blocks |
|:--|:--|:--|
| `check-class-a-gate` | Before any Write to `knowledge-base/class-a/` | Writes without a candidate file or librarian decision |
| `check-candidate-log-order` | Before any Write to `captures/candidates/` | Candidate files without a matching command log entry |
| `check-domain-index-before-class-a` | Before Class A write | Writes if domain index is stale from last round |
| `check-cycle-checkpoint-before-class-a` | Before Class A write | Writes if no checkpoint in last 5 entries |
| `block-git-exfiltration` | Before any Bash git command | Pushes of KB data to unauthorized remotes |

Exit code 2 = hard block. The tool call does not proceed.

---

## The Exit 127 Problem

If the hook runtime (Python) is not found, hooks exit with code 127.
Claude Code treats exit 127 as non-blocking — the tool call proceeds.

**This means all enforcement is silently bypassed if Python is not on PATH.**

The AWACS pre-flight check (`python3 .claude/check-hooks.py`) exists to catch this
before any pipeline work begins. All 28+ checks must pass. If any fail, stop.
The hooks ARE the pipeline. No hooks = no pipeline.

---

## Agent Roles

Five logical roles. Single-agent execution.

| Role | Responsibility | Writes To |
|:--|:--|:--|
| **Capture Runner** | Selects and executes commands, records raw output | `captures/command-log-*.jsonl` |
| **Analyzer** | Compares expected vs actual, prepares candidate files | `captures/candidates/` |
| **Librarian** | Runs gate, writes decisions, admits to Class A | `captures/librarian-decisions.jsonl`, `knowledge-base/class-a/` |
| **AIOS-Expert** | Supervisor check every 5 cycles, quality control | Read-only |
| **Get It Done** | General execution for non-pipeline tasks | Outside pipeline dirs |

Only the Librarian writes to `knowledge-base/class-a/`.
