# agent-governance

**Undo, provenance, and policy enforcement for AI agents — five Apache-2.0 tools that work alone and compose into one control plane.**

Agent security tooling answers *was this allowed* and *was this logged*. Almost none of it answers **can we take it back** — and that's the question that decides whether an incident costs an afternoon or a quarter.

```
containment = prevented + verifiably recovered
```

A system that detects every attack and reverses nothing scores 100% on detection and has prevented no loss.

## Start here

- **"I want to undo what my agent already did, or contain a compromised one"** → [revoco](https://github.com/rsh1k/revoco)
- **"I want a firewall in front of my agent's MCP tool calls"** → [mcp-gate](https://github.com/rsh1k/mcp-gate)
- **"I need to prove who authorized an action, cryptographically"** → [veritrail](https://github.com/rsh1k/veritrail)
- **"I want to protect what my agent remembers"** → [mnemosyne](https://github.com/rsh1k/mnemosyne)
- **"Just give me a terminal coding agent that does all this"** → [retie](https://github.com/rsh1k/retie)

## The five tools

| | What it is | Install |
|---|---|---|
| **[revoco](https://github.com/rsh1k/revoco)** ![stars](https://img.shields.io/github/stars/rsh1k/revoco?style=flat-square&label=%E2%98%85) | The control plane. Plans the rollback **before** the action runs, proves the rollback still works, and undoes a compromised grant's whole subtree in one call. 91 inverse specs across 8 surfaces (SAP, Workday, AWS, Entra/Okta, GitHub/K8s, databases, SaaS, filesystem/git). | `pip install revoco` ![pypi](https://img.shields.io/pypi/v/revoco?style=flat-square&label=) |
| **[retie](https://github.com/rsh1k/retie)** ![stars](https://img.shields.io/github/stars/rsh1k/retie?style=flat-square&label=%E2%98%85) | A terminal coding agent that gates on consequence: a sandboxed, network-cut shell is reversible so it runs without prompting; an unsandboxed one is irreversible so it asks a human. Tamper-evident ledger. Runs on hosted open models. | `pipx install retie` ![pypi](https://img.shields.io/pypi/v/retie?style=flat-square&label=) |
| **[mcp-gate](https://github.com/rsh1k/mcp-gate)** ![stars](https://img.shields.io/github/stars/rsh1k/mcp-gate?style=flat-square&label=%E2%98%85) | A drop-in policy firewall for MCP tool calls — allow / deny / require-approval / **redact on the wire** — with a hash-chained audit log. Wraps any stdio MCP server, zero runtime deps. | `pip install mcp-gate-policy` ![pypi](https://img.shields.io/pypi/v/mcp-gate-policy?style=flat-square&label=) |
| **[veritrail](https://github.com/rsh1k/veritrail)** ![stars](https://img.shields.io/github/stars/rsh1k/veritrail?style=flat-square&label=%E2%98%85) | Tamper-evident provenance as a service: Ed25519 attenuating delegation chains back to a named human, hijack detectors mapped to OWASP ASI 2026, forensic reports. FastAPI + SQLite/Postgres. | `pip install veritrail` ![pypi](https://img.shields.io/pypi/v/veritrail?style=flat-square&label=) |
| **[mnemosyne](https://github.com/rsh1k/mnemosyne)** ![stars](https://img.shields.io/github/stars/rsh1k/mnemosyne?style=flat-square&label=%E2%98%85) | A firewall for agent **memory**: provenance→trust tiers, a hard invariant that untrusted content never reaches an instruction surface, HMAC integrity at rest, re-scanned at read time. | `pip install mnemosyne-guard` ![pypi](https://img.shields.io/pypi/v/mnemosyne-guard?style=flat-square&label=) |

## 60 seconds

```bash
pip install revoco && revoco demo
```

Runs a vendor-payment-fraud scenario end to end: an agent subverts a rubber-stamping approver, wires money to a repointed bank account — then `cp.contain()` revokes the grant, rolls back everything done under it, and hands you a verifying hash-chained ledger. The residue (the remittance advice already sent) is reported, not hidden.

## Measured, not asserted

- **Containment benchmark** — 57 scenarios, 18 malicious / 39 benign, every attack technique paired with a benign twin on the same tools: **83.3% containment, 100% precision, 0 false positives**. Recovery is scored by comparing world state against a pre-attack baseline, never by trusting a rollback receipt — a phantom rollback that returns success while restoring nothing scores `UNCONTAINED`.
- **Real-filesystem drills** — the workstation adapter drilled against a real filesystem and a real git repo: **11/11 inverses restore state, 10/10 prose claims held** (inode reuse, mtime, broken hard links, `git reset --hard` irrecoverability — probed live).
- **Calibrated scanner** — threat scoring calibrated on 500 human-audited attack trajectories with task-disjoint splits: **AUROC 0.752**, reported honestly rather than as 0.97. Random splits inflate it by +8.4 points; that finding is documented, not hidden.

## How they fit together

```
retie (terminal coding agent)
  └── revoco (control plane: classify → enforce → plan the undo → record → verify)
        ├── mcp-gate    your agent's tool calls, if it speaks MCP
        ├── veritrail   provenance as a standalone service (FastAPI, Postgres)
        └── mnemosyne   what the agent remembers, treated as attack surface
```

revoco absorbs the cores of the other three; each satellite stays maintained for what it alone does — the stdio MCP proxy, the Postgres service and forensics module, the memory gateway.

## FAQ

**Why five repos and not one package?** Each tool is useful on its own, and installing one shouldn't drag in the others — firewalling a memory store shouldn't install 91 SAP/AWS inverse specs. The cores that genuinely overlap already merged into revoco; what remains in the satellites is the part with no revoco equivalent. This hub exists so you don't have to hold the map in your head.

**Which one do I start with?** If you're evaluating: `revoco demo` (30 seconds, no setup). If you're building: whichever single tool answers your problem today — they're independently released and don't require each other.

**Where do I report issues?** In the repo that owns the code. Cross-cutting questions — "which tool fits my setup" — are welcome in this repo's issues.

## Honest scope

One adapter surface (workstation) is validated against live systems; the other seven are specifications written from vendor documentation — work the [validation checklist](https://github.com/rsh1k/revoco/blob/main/docs/ADAPTERS.md) before trusting an inverse in production. A hash chain detects edits, reorders, and interior deletions but not truncation of the newest entries; anchor the head hash externally (`Ledger.checkpoint()` gives you the value to publish). Each repo opens with what it does and closes with what it doesn't — the READMEs undersell rather than oversell.

## News

- **2026-10-11** — suite-wide hygiene pass: NOTICE files added across all repos, private vulnerability reporting enabled everywhere, dependency advisories cleared (urllib3), stale CI PRs merged. Entries submitted to four awesome-lists, in review.

Apache-2.0 · Python 3.10+ · everything on PyPI
