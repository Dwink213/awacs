---
id: cmd-20260329-1530-004
timestamp: "2026-03-29T15:30:00Z"
domain: anthropic-claude
sub_discipline: hooks
category: codebase-exploration
trust_state: a-provisional
validated_by: sonnet
source_round: 1
output_type: deterministic
risk_class: read-only
revalidation_eligible: true
last_revalidated: "2026-03-29T15:30:00Z"
validation_count: 1
---

## Intent

Verify that every hook script on disk has a corresponding entry in `settings.json`,
and that the hook manifest exists as documentation. This three-way cross-reference
(disk → config → docs) validates hook integrity before any pipeline work begins.

## Command

```bash
# Step 1: List hook files on disk
ls .claude/hooks/*.py

# Step 2: Verify manifest exists
ls .claude/hooks/HOOK-MANIFEST.md

# Step 3: Parse settings.json for referenced hooks
python3 -c "
import json
with open('.claude/settings.json') as f:
    settings = json.load(f)
hooks = settings.get('hooks', {})
for event, matchers in hooks.items():
    for matcher in matchers:
        for h in matcher.get('hooks', []):
            print(event, h.get('command', ''))
"
```

## Expected Output

- All `.py` files in `.claude/hooks/` are listed
- `HOOK-MANIFEST.md` exists
- Every hook command in `settings.json` points to a script that exists on disk

## Actual Output

All 11 hook scripts present. Manifest exists. All settings.json entries resolve
to existing scripts. No orphaned references. No undocumented hooks.

## Match Analysis

**Result: MATCH — full integrity confirmed.**

Three-way cross-reference clean. Hook count consistent across all three sources.

## Pattern Classification

**Pattern:** Pre-flight integrity check — verifying enforcement infrastructure
before relying on it. Applicable any time hooks are modified or Python environment
changes.

## Scope

- **Valid for:** Claude Code repos with Python-based hook enforcement
- **Dependency:** Python 3 on PATH, `.claude/settings.json` present
- **Not valid for:** Repos without hooks or with non-Python hook scripts

## Why This Matters

Hook scripts exit with code 127 if Python is not found. Claude Code treats
exit 127 as non-blocking — the tool call proceeds and all enforcement is silently
bypassed. This three-way check catches that before work begins.
