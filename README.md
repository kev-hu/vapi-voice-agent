# EverSure Insurance — Inbound Voice Agent Prototype

A working voice agent prototype for EverSure Insurance, built on [Vapi](https://vapi.ai) to demonstrate AI-powered inbound call handling for an insurance contact center.

> **Context:** This prototype accompanies an engagement plan for a Voice AI pilot targeting 50%+ containment within 30 days of go-live.

## Try It

Open the [Vapi Dashboard](https://dashboard.vapi.ai), select the **Observe: EverSure Insurance** assistant, and start a test call. The mock customer is **Sarah Mitchell**, policy number **EVS-2024-1847362**.

## Architecture

```
Caller
  → Genesys Cloud (telephony / call routing)
    → Vapi Agent (voice AI)
        ├── Salesforce (customer verification, account data)
        ├── SharePoint via Azure AI Search (knowledge base)
        └── Genesys Cloud API (warm transfer to live agents)
```

In this prototype, integrations are simulated using Vapi **Code Tools** — TypeScript functions running on Vapi's infrastructure that return mock data. In production, these would be replaced by authenticated API calls to each system through a middleware layer.

## Intent Design

We scoped four intents based on EverSure's top call drivers. Each intent was classified by its **containment strategy** — whether the AI resolves it fully, or captures context and escalates.

| Intent | Strategy | Rationale |
|--------|----------|-----------|
| **Claim Status** | Full containment | Straightforward data lookup. Low risk — the agent reads back factual status, no judgment required. |
| **Payment Questions** | Containment + conditional escalation | Informational queries (balance, due date, how to pay) are safe to automate. Disputes and refunds require human judgment and audit trails, so they escalate to Billing Specialists. |
| **Coverage / Eligibility** | Always escalate | Providing specific coverage determinations carries E&O (errors & omissions) liability and requires a licensed agent in most states. The AI adds value by verifying identity and capturing the specific question before warm transfer — reducing AHT for the receiving agent. |
| **Address Updates** | Full containment | Simple write-back with read-back confirmation. Low risk, high automation value. |

### Why these four?

These map to EverSure's highest-volume call drivers. For a 6-week pilot targeting 50%+ containment, you want intents that are high-volume and have clear resolution paths. Claim status and address updates alone likely represent significant call volume with near-100% containability.

## Caller Verification

Identity verification happens **once per call** before any account-specific data is shared:

1. Policy number + last name
2. Verified against Salesforce (mock: `customer_lookup` tool)
3. Two attempts allowed, then escalate to a human

This balances security with caller experience — no one wants to re-verify for each question in the same call.

## Escalation Design

Escalation is not a failure mode — it's a feature. The agent is designed to escalate **confidently and with context**.

**When:**
- Payment disputes or refunds → Billing Specialists queue
- Coverage/eligibility questions → Licensed Agents queue (always)
- Caller requests a human → immediate, no friction
- Unrecognized intent or repeated confusion → General Support queue
- Caller frustration detected → General Support queue

**How (warm transfer):**
1. Agent tells the caller *why* they're being transferred
2. Agent passes a structured summary (identity verified, intent, conversation details) to the receiving queue
3. Caller doesn't repeat themselves

This was an intentional departure from Vapi's recommendation of "silent transfers." In insurance, transparency builds trust — callers should know why they're being connected and that their context is being carried over.

## Fallback & Uncertainty Handling

| Scenario | Agent Behavior |
|----------|---------------|
| Unrecognized intent | Clarify once: "Could you tell me a bit more about what you need?" |
| Still unclear after clarification | Escalate to General Support — don't loop |
| Tool/system failure | Apologize naturally, escalate immediately |
| Caller says "agent" or "representative" | Immediate transfer, no gatekeeping |
| Topic outside supported intents | Acknowledge limitation, transfer |

**Design principle:** The agent gets *one* chance to clarify. After that, a human is always better than a confused bot. This protects both caller experience and brand trust.

## Evaluation Framework

How to measure whether this agent meets the business goal:

| Metric | Target | What It Tells You |
|--------|--------|-------------------|
| **Containment Rate** | 50%+ within 30 days | % of calls resolved without human transfer. The primary success metric. |
| **Intent Recognition Accuracy** | 95%+ | Did the agent correctly identify what the caller wanted? Measured via transcript review. |
| **Escalation Rate by Intent** | Varies | Which intents need human help most? Informs future automation scope. |
| **Average Handle Time (AHT)** | < 6.5 min baseline | Is the AI faster than the current human average? |
| **AHT for Escalated Calls** | < baseline | Even when escalating, does pre-verification + context passing reduce the human agent's handle time? |
| **CSAT (Post-Call Survey)** | ≥ baseline | Caller satisfaction shouldn't drop. If it does, the agent needs tuning. |
| **False Containment Rate** | < 5% | Calls the AI "resolved" that the caller had to call back about. The most dangerous metric to ignore. |

### Coverage eligibility as an AHT-reduction play

Coverage/eligibility never counts toward containment — it always escalates. But it still delivers measurable value: the AI pre-verifies identity and captures the specific question, so the licensed agent gets a warm handoff instead of starting from scratch. This should reduce AHT on these calls by 1-2 minutes.

## Prompt Design

The system prompt (`prompt.md`) follows Vapi's recommended patterns:

- **Section-based structure:** Identity → Style → Response Guidelines → Intent flows → Escalation → Fallback → Closing
- **`<wait for user response>` markers:** Explicit turn-taking control so the agent doesn't steamroll the caller
- **Conditional branching:** Each intent has step-by-step flows with if/then logic
- **Voice-native formatting:** Numbers and dates spelled out for natural speech, no technical jargon
- **Tool abstraction:** The prompt references tools by name for the LLM, but includes strict rules to never mention tool names, functions, or system internals to the caller

## Repo Structure

```
├── README.md                        # This file
├── PROMPT.md                        # System prompt (source of truth)
├── assistant.json                   # Vapi assistant configuration (IDs redacted)
├── tools/
│   ├── customer_lookup.json         # Identity verification via Salesforce
│   ├── claim_status.json            # Claim status lookup
│   ├── payment_info.json            # Billing/payment information
│   ├── update_address.json          # Address update in Salesforce
│   └── transfer_to_agent.json       # Warm transfer via Genesys Cloud
└── analysis/
    ├── analysis-plan.json           # Post-call analysis configuration
    └── structured-outputs.json      # Structured data extraction schemas
```

## Production Considerations

Things that would change moving from prototype to production:

- **Code Tools → middleware API:** Replace mock data with authenticated calls to Salesforce REST API, Azure AI Search (SharePoint), and Genesys Cloud Platform API via a serverless middleware layer
- **Authentication:** OAuth 2.0 for Salesforce, Azure AD/Entra ID for SharePoint, Genesys OAuth for transfers
- **PII governance:** Middleware scopes returned fields — the LLM never sees SSNs, full account numbers, or other sensitive data it doesn't need
- **Audit logging:** Every tool call logged with call ID, timestamp, and data accessed for compliance
- **Call recording consent:** Managed at the Genesys Architect flow level before the call reaches the AI agent
- **Telephony:** Vapi connected to Genesys Cloud via SIP trunk for production call routing
