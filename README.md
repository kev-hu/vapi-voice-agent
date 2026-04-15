# Voice AI Pilot — Insurance Contact Center

I built this Voice AI agent prototype for an insurance contact center use case, along with the strategic plan to actually ship it.

The (fictional) client, **EverSure Insurance**, runs ~1.2M inbound calls/year at a 6.5-min AHT. The brief: ship a Voice AI pilot in 6 weeks, hitting **50%+ call containment within 30 days of go-live**.

What's in here: my scoping decisions, a working prototype, the ROI model, and the rollout plan I'd actually run.

### Built With

| Layer | Tool |
|-------|------|
| Voice AI orchestration | [Vapi](https://vapi.ai) |
| LLM | [OpenAI GPT-4o](https://openai.com) |
| Voice synthesis | [ElevenLabs](https://elevenlabs.io) |
| Transcription | [Deepgram](https://deepgram.com) |
| Integrations *(mocked)* | Salesforce (CRM) · Genesys Cloud (CCaaS) · SharePoint via Azure AI Search (KB) |

---

## How I Approach This Work

This is how I'd run any AI build, in any domain. The artifact happens to be an insurance contact center; the approach travels.

1. **Talk to people. Build the ROI story.** Before anything gets built, I spend time with the people who do the work, the people affected by it, and the people who have to sign off (compliance, legal, IT). The output is a clear ROI story: who saves time, how much, what we'd give up, what could go wrong. If I can't write that story compellingly, nothing else matters yet.

   *Concrete example from this case:* I wouldn't have known the first message has to include an AI disclosure *and* recording consent in a specific order, or that coverage determinations are E&O liability the AI literally can't touch in most states. None of that is in the Vapi docs. You learn it by researching the domain and sitting with the compliance lead in week one.

2. **Prototype to find the shape.** Once the narrative holds, I build the smallest version that proves it works — core prompts, intents, routes, escalation paths. The goal isn't polish; it's to surface the things you can only learn by trying (the warm-voice problem, the edge cases, where the integrations break).

3. **Plan the measurement, then roll it out.** Phased traffic (10% → 100%), instrumented from day one. The metric I care about most isn't containment — it's *false* containment (calls AI "resolved" that the caller had to call back about). Without that number, every other number is a vanity metric.

4. **Enabling success for humans.** FAQ for the team adopting it, ops runbook for live incidents, internal training, a post-deploy retro that actually feeds back into the prompt. This is where a working pilot becomes something the org can own without me there.

---

## 🎯 The Strategic Call — Picking the Right Intents

EverSure's top four call drivers are claim status, payment questions, coverage/eligibility, and address updates. The obvious move is to go after **coverage/eligibility** first — it's the longest, most painful call type.

I think that's wrong.

| Intent | Volume | Complexity | Containment Potential | Pilot? |
|--------|--------|------------|------------------------|--------|
| **Address Updates** | Very High | Low | Very High (CRM write) | ✅ Yes |
| **Claim Status** | Medium | Low | High (CRM read) | ✅ Yes |
| **Payment Questions** | Medium | Medium | Medium (with escalation) | ⚠️ Phase 2 |
| **Coverage / Eligibility** | Medium-High | High | Low (human judgment) | ❌ No |

**Why I called it that way:** coverage/eligibility carries E&O (errors & omissions) liability and requires a licensed agent in most states. Automating it is high-risk, low-reward. Address updates and claim status are the opposite — high-volume, low-complexity, clean resolution paths. That's where you hit your containment number.

**What you get:** faster time-to-value on the pilot, *and* coverage/eligibility AHT still drops because the AI handles identity verification and captures the specific question before warm-transferring. Containment win on the easy intents, AHT win on the hard one — without the liability.

This was the call I'd defend the hardest in front of the exec sponsor.

---

## 📞 Demo

> **Video walkthrough:** _I'll add a recording soon — a live call covering claim status, an address update, and a coverage/eligibility escalation._

For specific scenarios, the **[sample call transcripts](demo/sample-call.md)** walk through the happy path, an escalation, and a failed verification.

Want to try the live agent? Reach out on [LinkedIn](https://www.linkedin.com/in/kevinqinhu/).

---

## 🤖 Agent Design & Prompts

I designed identity verification to happen **once per call**, before any account data gets shared (policy # + last name → Salesforce, two attempts, then escalate). Verifying once is a deliberate trade-off: security still holds, and no one wants to re-auth for every question in the same conversation.

I treated **escalation as a first-class feature**, not a failure mode. Every transfer is warm — the receiving human gets identity status, intent, and a conversation summary so the caller never repeats themselves. Triggers: payment disputes, any coverage/eligibility question, caller frustration, repeated confusion, or any explicit ask for a human.

**The agent gets one chance to clarify.** After that, a human is always better than a confused bot. This is what protects caller experience and prevents the dead-end loops that kill CSAT in most voice AI deployments.

The prompt follows Vapi's section-based pattern (Identity → Style → Response Guidelines → Intent flows → Escalation → Fallback → Closing) with explicit `<wait for user response>` markers, voice-native formatting (numbers and dates spelled out), and hard rules against ever surfacing tool names or system internals to the caller. Full prompt in [`PROMPT.md`](PROMPT.md).

The non-obvious lessons from writing it (warm voice ≠ right voice, why section-based beats narrative, what most people get wrong about turn-taking) are in **[`case-study/prompt-insights.md`](case-study/prompt-insights.md)**.

---

## 💰 ROI

I modeled Phase 1 only (Claim Status + Address Updates, 660K calls/year):

- **Annual savings:** $2.27M (48% cost reduction)
- **FTE-equivalents freed:** 10.6
- **AHT savings on contained calls:** 4 min (6.5 → 2.5)

Full inputs, sensitivity, and the live spreadsheet in **[`demo/roi-calculator.md`](demo/roi-calculator.md)**.

---

## 🏗️ Architecture

```
Caller
  → Genesys Cloud (telephony / call routing)
    → Vapi Agent (voice AI)
        ├── Salesforce (customer verification, account data)
        ├── SharePoint via Azure AI Search (knowledge base)
        └── Genesys Cloud API (warm transfer to live agents)
```

For this prototype, I simulated the integrations using Vapi **Code Tools** — TypeScript functions running on Vapi's infrastructure that return mock data. In production, those become authenticated API calls through a middleware layer.

### Tools

| Tool | Purpose |
|------|---------|
| `customer_lookup` | Identity verification (policy # + last name) → Salesforce |
| `claim_status` | Claim lookup by claim number → Salesforce |
| `payment_info` | Billing/payment details → Salesforce |
| `update_address` | Update mailing address → Salesforce |
| `transfer_to_agent` | Warm transfer with context → Genesys Cloud |

---

## 💭 What I'd Do Differently

Honest reflections after building this:

- **Start with transcripts, not architecture.** I designed top-down (architecture → tools → prompt). I'd flip it next time — write 20-30 sample call transcripts first, the happy paths and the weird edge cases, and let the prompt fall out of the conversation patterns I actually want. The prompt I wrote handles the cases I imagined, not the cases real callers create.
- **Mock data is a crutch.** The mock tools return clean, perfect Salesforce records. Real Salesforce data is messy — missing fields, inconsistent formatting, partial info. The agent looks better in this prototype than it would on day one in production. The first thing I'd do with real data access is stress-test against the dirty 20%.
- **Pass/fail eval is the floor, not the ceiling.** The current analysis plan does basic summary + pass/fail per call. To actually validate in production, I'd build a structured eval set — 100+ historical calls re-run against the agent and scored on intent recognition, identity-verification compliance, escalation appropriateness, and false containment. Without that, "containment rate" is a vanity metric.
- **Voice choice deserves an A/B test.** I picked ElevenLabs by default. Voice prosody, latency, and "feel" affect containment as much as prompt design does. I should have tested two or three voices on a panel of real call recordings before locking it in.

---

## 📚 Dig Deeper

| File | What's in it |
|------|--------------|
| [`demo/sample-call.md`](demo/sample-call.md) | Three sample call transcripts — happy path, escalation, failed verification |
| [`PROMPT.md`](PROMPT.md) | The full system prompt — every section, intent flow, and guardrail |
| [`case-study/prompt-insights.md`](case-study/prompt-insights.md) | Hard-earned lessons from writing the prompt (compliance, voice register, turn-taking) |
| [`demo/roi-calculator.md`](demo/roi-calculator.md) | The ROI model — inputs, calc, sensitivity, with the live spreadsheet screenshot |
| [`case-study/engagement-detail.md`](case-study/engagement-detail.md) | The full engagement plan — discovery, timeline, phased ramp, eval framework, risks |

---

## Repo Structure

```
├── README.md                        # This file — case study overview
├── LICENSE                          # MIT
├── PROMPT.md                        # System prompt (source of truth)
├── assistant.json                   # Vapi assistant config (IDs redacted)
├── tools/                           # Vapi Code Tools (mock data)
│   ├── customer_lookup.json
│   ├── claim_status.json
│   ├── payment_info.json
│   ├── update_address.json
│   └── transfer_to_agent.json
├── analysis/                        # Post-call analysis configuration
│   ├── analysis-plan.json
│   └── structured-outputs.json
├── demo/
│   ├── sample-call.md               # Sample call transcripts
│   ├── roi-calculator.md            # ROI calculator (with screenshot)
│   └── roi-calculator.png
└── case-study/
    ├── engagement-detail.md         # Full engagement plan, evaluation, risks, production
    └── prompt-insights.md           # Non-obvious lessons from writing the system prompt
```

---

**Author:** Kevin Hu · [LinkedIn](https://www.linkedin.com/in/kevinqinhu/)
