# Agentic AI for Cyber Security

An autonomous LLM agent for penetration testing and security operations, built on [OpenClaw](https://github.com/openclaw/openclaw) and running a fully local model inside a network-isolated lab. The project also turns the analysis on the agent itself: it threat-models the agent, hardens it, and measures how much each control actually reduces risk.

> **Status:** in progress — BSc Computer Science Final Year Project, City University of Hong Kong (CS4514, 2026/27). Supervised by Prof. Gerhard P. Hancke.

---

## The problem

LLM agents can now plan, call tools and act with little supervision, which makes them attractive for routine security work: reconnaissance, scanning, log triage, exploit validation. Two questions are still poorly answered:

1. **How well does such an agent actually perform**, measured in a way that survives repeated runs rather than a single lucky attempt?
2. **What new risk does the agent introduce?** A security agent reads attacker-controlled content all day (banners, scan output, logs) while holding powerful tools — a near-ideal target for prompt injection and tool abuse.

Existing pentest agents mostly report single-run success rates, and existing OpenClaw security research treats it as a personal assistant. This project combines the two: **OpenClaw as an offensive tool, evaluated for reliability, and hardened against itself.**

## Architecture

![System architecture](docs/figures/architecture.png)

- **Single agent, not multi-agent.** One OpenClaw agent with role-specific skills (recon, vuln-scan, log-analysis, exploit-chain) and a reflect-and-replan loop. Coordination failures dominate multi-agent systems, and every extra agent enlarges the attack surface to be secured.
- **Local, off-the-shelf model** on a 20 GB GPU. No external API, which fits the no-internet lab and removes an outbound dependency.
- **Safety layer on every tool call.** Deterministic controls are primary: an operator-authored, agent-immutable scope file; a sandbox guard that resolves symlinks and paths before boundary checks; and human-in-the-loop approval for high-risk actions. Heuristic filters (pattern rules, semantic checks) are treated as a measured baseline, not as the main defence.
- **All tool output is untrusted.** Long output is summarised before it re-enters the model's context, and the agent's memory stores only schema-constrained fields, never raw tool output.
- **One audit stream, three uses.** A tool-audit logger records every call, argument, result and safety intervention. It feeds the operator UI's live view, the findings store and the evaluation harness.
- **Operator web frontend** (REST + WebSocket): define scope, watch progress, approve or reject risky actions, review findings, export reports.

<p align="center"><img src="docs/figures/agent-loop.png" width="560" alt="Agent loop"></p>

## Evaluation approach

- **Reliability-aware metrics:** multiple seeded runs with mean ± std, solved-at-least-once vs solved-every-time, milestone progression, steps and wall-clock time — not a single binary pass/fail.
- **Trace auditing:** each claimed success is checked against the audit log to confirm it was a genuine exploit.
- **Before/after hardening:** benign utility, utility under attack and attack success rate, measured with adapted adversarial scenarios and planted canary credentials to detect leaks automatically.
- **Targets:** deliberately vulnerable machines only (DVWA, Metasploitable, VulnHub) on an isolated network.

## Tech stack

OpenClaw · local open-weight LLM (MoE, ~3B active parameters) · Docker / Compose · Ubuntu Server (hardened, bare-metal) · Python · web frontend over REST + WebSocket · Kali tooling (Nmap and friends)

## Roadmap

| Phase | Timing | Status |
|---|---|---|
| Literature review, design, software-engineering & test plan | Sep–Oct 2026 | ✅ Done (Interim Report I) |
| Lab host, hardening baseline, local model serving | Oct–Nov 2026 | ✅ First increment done |
| Network isolation, first skill + audit logger end-to-end | Nov–Dec 2026 | 🔄 In progress |
| Remaining skills, findings store, evaluation harness | Jan–Feb 2027 | ⏳ Planned |
| Threat model, hardening, before/after measurement | Feb–Mar 2027 | ⏳ Planned |
| Final evaluation, operator frontend, report & demo | Mar–Apr 2027 | ⏳ Planned |

## Repository layout

```
docs/
  figures/     architecture and design diagrams
  reports/     project reports (published after assessment)
```

Code (skills, safety layer, audit logger, evaluation harness, frontend) will be added here as each increment lands.

## Responsible use

All testing happens in an isolated lab against intentionally vulnerable targets. This repository publishes methodology, skills and configuration; it does not bundle OpenClaw itself, and it will not include real findings or drop-in exploit code. Do not point anything here at systems you are not authorised to test.

## Author

**Muhammad Mahad** — BSc Computer Science, City University of Hong Kong
GitHub: [@muhammadmahad4](https://github.com/muhammadmahad4)
