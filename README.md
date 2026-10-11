# agent-governance

**Undo, provenance, and policy enforcement for AI agents — five Apache-2.0 tools that work alone and compose into one control plane.**

Agent security tooling answers *was this allowed* and *was this logged*. Almost none of it answers **can we take it back** — and that's the question that decides whether an incident costs an afternoon or a quarter.

```
containment = prevented + verifiably recovered
```

A system that detects every attack and reverses nothing scores 100% on detection and has prevented no loss.

## The five tools

| | What it is | Install |
|---|---|---|
| **[revoco](https://github.com/rsh1k/revoco)** | The control plane. Plans the rollback **before** the action runs, proves the rollback still works, and undoes a compromised grant's whole subtree in one call. 91 inverse specs across 8 surfaces (SAP, Workday, AWS, Entra/Okta, GitHub/K8s, databases, SaaS, filesystem/git). | `pip install revoco` |
| **[retie](https://github.com/rsh1k/retie)** | A terminal coding agent that gates on consequence: a sandboxed, network-cut shell is reversible so it runs without prompting; an unsandboxed one is irreversible so it asks a human. Tamper-evident ledger. Runs on hosted open models. | `pipx install retie` |
| **[mcp-gate](https://github.com/rsh1k/mcp-gate)** | A drop-in policy firewall for MCP tool calls — allow / deny / require-approval / **redact on the wire** — with a hash-chained audit log. Wraps any stdio MCP server, zero runtime deps. | `pip install mcp-gate-policy` |
| **[veritrail](https://github.com/rsh1k/veritrail)** | Tamper-evident provenance as a service: Ed25519 attenuating delegation chains back to a named human, hijack detectors mapped to OWASP ASI 2026, forensic reports. FastAPI + SQLite/Postgres. | `pip install veritrail` |
| **[mnemosyne](https://github.com/rsh1k/mnemosyne)** | A firewall for agent **memory**: provenance→trust tiers, a hard invariant that untrusted content never reaches an instruction surface, HMAC integrity at rest, re-scanned at read time. | `pip install mnemosyne-guard` |

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

## Honest scope

One adapter surface (workstation) is validated against live systems; the other seven are specifications written from vendor documentation — work the [validation checklist](https://github.com/rsh1k/revoco/blob/main/docs/ADAPTERS.md) before trusting an inverse in production. A hash chain detects edits, reorders, and interior deletions but not truncation of the newest entries; anchor the head hash externally (`Ledger.checkpoint()` gives you the value to publish). Each repo opens with what it does and closes with what it doesn't — the READMEs undersell rather than oversell.

Apache-2.0 · Python 3.10+ · everything on PyPI
