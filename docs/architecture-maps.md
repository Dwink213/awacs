# AWACS Pipeline — Architecture Maps

> Four Mermaid diagrams showing how the pipeline enforces itself.
> Conceptual view — methodology only. Implementation details stay private.

Paste any diagram into [mermaid.live](https://mermaid.live) to view it rendered.
Start with Diagram 1 — it's the core contract. Everything else supports that path.

---

## Written Analysis

The AWACS pipeline is a **file-mediated, enforcement-gated knowledge system**. Agents never call each other directly — all communication flows through well-known file paths. This makes the dependency graph a *data flow graph*, not a call graph: every connection is a file read or write, and every step is verifiable after the fact.

The hook layer is the integrity guarantee. Every write to the knowledge base is blocked at the tool level unless all prior steps are verifiable in the file system. The agents are the actors; the hooks are the referees.

Session loops operate at two levels: always-on monitors that run on a fixed cadence regardless of mode, and burst loops that activate dynamically when pipeline signals indicate activity.

---

## Diagram 1 — The Write Chain with Enforcement Gates

The critical path from command execution to a Class A knowledge base entry. Every arrow that crosses a gate has a hook blocking it at the tool level.

```mermaid
flowchart TD
    classDef agent fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef file fill:#fef9c3,stroke:#ca8a04,color:#713f12
    classDef hook fill:#fce7f3,stroke:#db2777,color:#831843
    classDef kb fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef block fill:#fee2e2,stroke:#dc2626,color:#7f1d1d

    EXEC["🖥️ Command Executes\n(Capture Agent)"]:::agent
    LOG[("📄 Command Log\n/captures/")]:::file

    EXEC -->|"Post-execution hook\ntamper detection"| HASH["🔒 Output Hash Recorded\nsha256 — tamper detection"]:::hook
    HASH --> LOG

    LOG --> ANA["🔍 Analysis Agent\nClassifies patterns"]:::agent

    GATE1{"Hook Gate 1\nOrdering check"}:::hook
    ANA -->|"Pre-write hook"| GATE1
    GATE1 -->|"❌ BLOCK"| BLOCK1["🚫 Write Blocked\nexit(2)"]:::block
    GATE1 -->|"✅ PASS"| CAND[("📝 Candidate File\n/captures/candidates/")]:::file

    CAND --> LIB["📚 Governance Agent\nAdmission gate"]:::agent
    LIB --> DEC[("⚖️ Decision Log\n/captures/")]:::file

    GATE2{"Hook Gate 2\nAdmission check"}:::hook
    GATE3{"Hook Gate 3\nIndex currency check"}:::hook
    GATE4{"Hook Gate 4\nCheckpoint check"}:::hook

    LIB -->|"Pre-write hook\nClass A target"| GATE2
    GATE2 -->|"❌ BLOCK — no admit"| BLOCK2["🚫 Write Blocked\nexit(2)"]:::block
    GATE2 -->|"✅ PASS"| GATE3
    GATE3 -->|"❌ BLOCK — stale index"| BLOCK3["🚫 Write Blocked\nexit(2)"]:::block
    GATE3 -->|"✅ PASS"| GATE4
    GATE4 -->|"❌ BLOCK — no checkpoint"| BLOCK4["🚫 Write Blocked\nexit(2)"]:::block
    GATE4 -->|"✅ PASS"| CLASSA[("✅ knowledge-base/class-a/\ndomain/subdomain/entry.md")]:::kb

    CLASSA --> IDX[("📑 Domain Index\n/knowledge-base/indexes/")]:::kb
    IDX --> TOC[("📖 Master TOC\n/knowledge-base/indexes/")]:::kb
    TOC --> CHKPT[("🏁 Cycle Checkpoint\n/captures/")]:::file

    CHKPT -.->|"Post-write hooks\nreminders + logging"| REM["📣 Reminder hooks\nAudit event hooks"]:::hook
```

---

## Diagram 2 — Agent Roles & Escalation Paths

Who reads and writes what, the supervisor cadence, and the escalation triggers that break normal cycle flow.

```mermaid
flowchart TB
    classDef supervisor fill:#f3e8ff,stroke:#7c3aed,color:#3b0764
    classDef runner fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef analyzer fill:#fff7ed,stroke:#ea580c,color:#7c2d12
    classDef librarian fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef file fill:#fef9c3,stroke:#ca8a04,color:#713f12
    classDef trigger fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef skill fill:#e0e7ff,stroke:#4338ca,color:#1e1b4b

    subgraph SUPERVISOR["🔭 Supervisor Layer — periodic check-in"]
        EXP["Supervision Agent\nRead-only quality control\nPattern recognition"]:::supervisor
        SRPT[("Supervisor Reports\n/captures/")]:::file
    end

    subgraph PIPELINE["⚙️ Pipeline Cycle"]
        direction LR
        RUN["Capture Agent\nExecutes commands"]:::runner
        ANA["Analysis Agent\nClassifies patterns"]:::analyzer
        LIB["Governance Agent\nAdmission gate"]:::librarian
    end

    subgraph FILES["💾 File System — the communication medium"]
        direction TB
        CLOG[("Command Log")]:::file
        CAND[("Candidate Files")]:::file
        LDEC[("Decision Log")]:::file
        CLSA[("knowledge-base/class-a/")]:::file
        PARK[("Parking Lot")]:::file
        DRIFT[("Drift Events")]:::file
        CHKPT[("Cycle Checkpoints")]:::file
    end

    subgraph ESCALATION["⚠️ Supervisor Escalation Triggers"]
        T1["N consecutive rejections"]:::trigger
        T2["Repeated error pattern"]:::trigger
        T3["High-cost model escalation rate"]:::trigger
        T4["Critical drift event"]:::trigger
        T5["Parking lot spike"]:::trigger
        T6["Pipeline tunnel vision signal"]:::trigger
        OPUSESC["Analysis escalation flag\nReasoning limit reached"]:::trigger
    end

    RUN -->|"writes"| CLOG
    CLOG -->|"reads"| ANA
    ANA -->|"writes"| CAND
    CAND -->|"reads"| LIB
    LIB -->|"writes"| LDEC
    LIB -->|"writes"| CLSA
    LIB -->|"writes"| DRIFT
    LIB -->|"writes"| CHKPT
    LIB -->|"parks failed"| PARK

    EXP -->|"reads all"| CHKPT
    EXP -->|"reads"| CLSA
    EXP -->|"reads"| PARK
    EXP -->|"reads"| DRIFT
    EXP -->|"writes"| SRPT
    EXP -.->|"can redirect"| RUN
    EXP -.->|"can redirect"| ANA
    EXP -.->|"can redirect"| LIB

    T1 & T2 & T3 & T4 & T5 & T6 -->|"immediate trigger"| EXP
    ANA -->|"flags"| OPUSESC
    OPUSESC -.->|"escalates to"| EXP
```

---

## Diagram 3 — Session Loops: Cadence & Activation

Always-on monitors and signal-activated burst loops. The Session Controller reads the file system to determine state — not in-memory signals.

```mermaid
flowchart TD
    classDef always fill:#f3e8ff,stroke:#7c3aed,color:#3b0764
    classDef burst fill:#fff7ed,stroke:#ea580c,color:#7c2d12
    classDef idle fill:#f1f5f9,stroke:#64748b,color:#334155
    classDef governor fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef state fill:#fef9c3,stroke:#ca8a04,color:#713f12
    classDef signal fill:#dcfce7,stroke:#16a34a,color:#14532d

    subgraph ALWAYS["🟢 Always-On — run regardless of mode"]
        WAT["👁️ Health Monitor\nFixed cadence\nDetects stale background processes"]:::always
        GOV["🎛️ Session Controller\nFixed cadence\nReads signals → sets mode\nUpdates session state file"]:::governor
    end

    subgraph STATE["📋 Session State\n(file-based)"]
        MODE["Active mode"]:::state
        WKON["Current context"]:::state
        LOOPS["Loop status map"]:::state
        LGOV["Last controller run timestamp"]:::state
    end

    subgraph SIGNALS["📡 Pipeline Signals — Controller reads"]
        SIG1["Candidate queue depth"]:::signal
        SIG2["Command log recent activity"]:::signal
        SIG3["Knowledge base growth rate"]:::signal
        SIG4["Maintenance check staleness"]:::signal
    end

    subgraph BURST["🟡 Burst Loops — Controller activates"]
        LIB_LOOP["📚 Admission Loop\nActivates: queue depth exceeds threshold"]:::burst
        IC_LOOP["🖥️ Capture Loop\nActivates: log active AND queue > 0"]:::burst
        REL_LOOP["🔍 Source Loop\nActivates: source-scan mode"]:::burst
        TEST_LOOP["🧪 Validation Loop\nActivates: per source found"]:::burst
    end

    subgraph ONDEMAND["⚪ On-Demand"]
        SWEEP["Pipeline sweep\nSession start + on-demand"]:::idle
        PREFLIGHT["Pre-flight check\nMandatory before pipeline work"]:::idle
        DRIFT["Drift detection\nOn-demand"]:::idle
        STATS["Pipeline stats\nOn-demand"]:::idle
        REVAL["Trust revalidation\nOn-demand"]:::idle
    end

    GOV -->|"reads"| SIGNALS
    SIGNALS --> GOV
    GOV -->|"writes"| STATE
    GOV -->|"threshold met"| LIB_LOOP
    GOV -->|"log active + queue"| IC_LOOP
    GOV -->|"source-scan mode"| REL_LOOP
    REL_LOOP -->|"on each find"| TEST_LOOP
    WAT -.->|"health status"| GOV
    LIB_LOOP -.->|"feeds back into"| SIG1
    IC_LOOP -.->|"feeds back into"| SIG2
```

---

## Diagram 4 — Hook Firing Sequence

Which hook categories fire on which tool events, and whether they block or allow the operation. Exit 2 = hard block. Exit 0 = allow. Exit 127 = runtime missing = non-blocking (see pre-flight requirement).

```mermaid
flowchart LR
    classDef event fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef hook fill:#fce7f3,stroke:#db2777,color:#831843
    classDef block fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef pass fill:#dcfce7,stroke:#16a34a,color:#14532d
    classDef log fill:#fef9c3,stroke:#ca8a04,color:#713f12

    subgraph PRE_WRITE["PreToolUse — Write / Edit (fires BEFORE)"]
        direction TB
        EV1["✍️ Write or Edit\ntool called"]:::event
        H1["Safety Gate\nEnvironment protection"]:::hook
        H2["Build Order Gate 1\nDocument-before-code"]:::hook
        H3["Build Order Gate 2\nTest-before-stack"]:::hook
        H4["Build Order Gate 3\nTest-before-advance"]:::hook
        H5["Hook Gate 1\nOrdering enforcement"]:::hook
        H6["Hook Gate 2\nIndex currency"]:::hook
        H7["Hook Gate 3\nCheckpoint recency"]:::hook
        H8["Hook Gate 4\nAdmission decision"]:::hook
        PASS1["✅ Write Allowed"]:::pass
        BLOCK1["🚫 exit(2) — Write Blocked"]:::block

        EV1 --> H1 --> H2 --> H3 --> H4 --> H5 --> H6 --> H7 --> H8
        H8 -->|"all pass"| PASS1
        H1 & H2 & H3 & H4 & H5 & H6 & H7 & H8 -->|"any fail"| BLOCK1
    end

    subgraph PRE_BASH["PreToolUse — Bash (fires BEFORE)"]
        direction TB
        EV2["💻 Bash tool called"]:::event
        H9["Exfiltration Guard\nBlocks unauthorized data export"]:::hook
        H10["⛔ Destructive Command BRAKE\nBlocks dangerous operations"]:::hook
        PASS2["✅ Bash Allowed"]:::pass
        BLOCK2["🚫 exit(2) — Command Blocked"]:::block

        EV2 --> H9 --> H10
        H10 -->|"safe"| PASS2
        H9 & H10 -->|"dangerous"| BLOCK2
    end

    subgraph POST_WRITE["PostToolUse — Write (fires AFTER)"]
        direction TB
        EV3["✅ Write completed"]:::event
        H11["Reminder Hook A\nCapture discipline"]:::log
        H12["Reminder Hook B\nCheckpoint discipline"]:::log
        H13["Audit Hook\nAdmission event log"]:::log

        EV3 --> H11 --> H12 --> H13
    end

    subgraph POST_BASH["PostToolUse — Bash (fires AFTER)"]
        direction TB
        EV4["✅ Bash completed"]:::event
        H14["Execution Audit Hook\nOutput hash — tamper detection"]:::log

        EV4 --> H14
    end

    subgraph SESSION_EVENTS["Session Lifecycle"]
        direction TB
        EV5["🚀 SessionStart"]:::event
        EV6["🗜️ PostCompact"]:::event
        H15["Startup Hook\nPending work sweep"]:::hook
        H16["Compaction Hook\nContext reorientation"]:::hook

        EV5 --> H15
        EV6 --> H16
    end
```

---

## Key Observations

**1. The write chain is a one-way ratchet.**
Data only flows forward. There is no path to write Class A that bypasses any gate. The hooks enforce this at the tool level, not by convention.

**2. The Session Controller is the only component with write access to session state.**
All other loops are reactive. The Controller is the brain; burst loops are the arms.

**3. Escalation is pull, not push.**
The Supervision Agent reads the file system on cadence and infers state from counts and timestamps. It can be run at any time and will produce an accurate picture from files alone.

**4. The Exit 127 problem is the single point of failure.**
Every hook depends on the Python runtime. If it's missing, exit 127 is non-blocking and the entire enforcement layer silently vanishes. The pre-flight check exists entirely to catch this before it happens.

**5. In a quiet session, only two processes run.**
The Health Monitor and Session Controller run every 30 minutes. Everything else is idle until signals activate it.

---

[← Write chain spec](write-chain.md) · [Trust tier rules →](trust-tier-rules.md)
