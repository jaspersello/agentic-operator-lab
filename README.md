# agentic-operator-lab

A two-week sprint building and shipping production-style **agentic workflows** — designing AI agents that execute real business processes with human oversight, reliability guardrails, and observability.

The goal: demonstrate the **"PM brain + engineer hands"** profile — turning SOPs into executable agent playbooks, wiring tools and data, setting human-in-the-loop guardrails, and iterating on outcomes.

## Stack
`n8n` (orchestration) · `Claude` (reasoning + context engineering) · `Supabase / pgvector` (memory + RAG) · markdown skill libraries (the "company brain") · HITL gates · LLM-as-judge evals

## Projects
| # | Project | Domain | Status |
|---|---------|--------|--------|
| 1 | Lead enrichment + personalized outreach drafter | Growth / RevOps | 🟡 In progress |
| 2 | Intelligent ticket triage + RAG-grounded SOP executor | Ops / CS | ⚪ Planned |
| 3 | Evals + guardrails framework (LLM-as-judge, validation, PII redaction) | Reliability | ⚪ Planned |

## Repo structure
```
agentic-operator-lab/
├── skills/                  # the "company brain" — markdown context files
│   ├── brand-voice.md
│   ├── lead-qualification-sop.md
│   └── outreach-playbook.md
├── workflows/               # versioned n8n workflow exports
│   └── hello-agent.json
└── README.md
```

## Day 1 — Hello Agent
A minimal lead-qualification agent: a sample lead flows through Claude, which scores it (Tier A/B/C) and drafts a first-touch message in brand voice, returning structured JSON. Built as the foundation the Week-1 project extends.

See `workflows/hello-agent.json`. Setup steps in the sprint's `DAY-1-SETUP.md`.

---
*Built as a focused upskilling sprint. Each workflow ships with an architecture note and a short demo.*
