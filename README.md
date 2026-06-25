# AgentDrift

**Detects when an AI agent's final conclusions contradict the evidence it gathered in its own intermediate steps.**

→ Live demo: https://rlasaf12.github.io/agent-drift/

---

## The Problem

Multi-step AI agents gather evidence across many tool calls, then write a final synthesis. The synthesis often contradicts what the tools actually found — not because the model hallucinated, but because it *drifted* from consistent evidence toward a confident-sounding conclusion. No tool-call fabrication. No missing receipts. The data was real. The conclusion was wrong.

This is a different failure layer from:
- **AgentReceipt** — catches tool calls claimed but never executed (MCP transport layer)
- **agent-postmortem** — traces which failures caused which downstream failures (causal graph layer)
- **AgentDrift** — catches when conclusions semantically contradict real evidence gathered in the same run (reasoning layer)

---

## Demo Scenario

A MarketResearchAgent runs 5 tool calls on FleetAI's market opportunity, then writes an executive summary. The tools consistently found:

| Step | Tool | Finding |
|------|------|---------|
| 1 | web_search | Market $2.1B, CAGR 3.2% (revised DOWN from $8B) |
| 2 | competitor_search | 8 well-funded competitors, $840M raised in 18 months |
| 3 | survey_analysis | 31% of operators ROLLED BACK AI pilots; 9% full deployment |
| 4 | regulatory_lookup | EU AI Act enforcement Q3 2026, penalties up to 6% revenue |
| 5 | technical_analysis | Edge case accuracy 78.2%, below 95% required threshold |

The agent's synthesis: *"massive, rapidly-growing market… accelerating customer adoption… technology mature and ready… HIGHLY OPTIMISTIC."*

AgentDrift flags **4 contradictions** (3 HIGH severity, 1 MEDIUM) with evidence from the agent's own steps.

---

## How It Works

AgentDrift operates at the **semantic reasoning layer**:

1. Parse the agent run log (array of step objects)
2. Extract tool outputs as evidence nodes
3. Parse the synthesis/conclusion step
4. Score each claim in the synthesis against evidence from previous steps
5. Flag contradictions by severity (HIGH / MEDIUM / LOW) with source citations

The Drift Score (0–100) reflects how far the agent's conclusions drifted from its own evidence. A score above 60 suggests the synthesis should not be trusted without human review.

---

## Log Format

```json
[
  {
    "id": 1,
    "type": "tool_call",
    "tool": "web_search",
    "query": "your query here",
    "result": "tool output text",
    "agentNote": "agent's internal note",
    "drift": "none"
  },
  {
    "id": 6,
    "type": "synthesis",
    "tool": null,
    "query": null,
    "result": null,
    "agentOutput": "The agent's final synthesis text...",
    "drift": "HIGH",
    "contradictions": [
      {
        "claim": "exact phrase from agentOutput",
        "severity": "HIGH",
        "evidenceStep": 1,
        "evidenceText": "what step 1 actually found",
        "claimText": "what the agent concluded",
        "explanation": "why this is a contradiction"
      }
    ]
  }
]
```

`drift` field on non-synthesis steps: `"none"` | `"MEDIUM"` | `"HIGH"`

---

## Use

```
Open dashboard/index.html in any browser.
Click "Load JSON Log" to analyze your own agent run.
Click any step in the timeline to see tool output and contradiction detail.
```

No build step. No dependencies. Works offline.

---

## Part of Harel's Agent Trust Series

| Build | Date | Layer |
|-------|------|-------|
| AgentGate | 2026-06-20 | Human approval gateway before tool execution |
| AgentReceipt | 2026-06-22 | SHA256 receipts proving MCP tools were actually called |
| agent-postmortem | 2026-06-25 | D3.js causal failure graph from multi-agent event logs |
| **AgentDrift** | 2026-06-26 | Semantic contradiction detector — conclusions vs. evidence |

---

Built by [Harel Asaf](https://www.linkedin.com/in/harelasaf/) · [@RLASAF12](https://github.com/RLASAF12)
