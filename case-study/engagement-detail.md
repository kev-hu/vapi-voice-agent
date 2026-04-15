# Engagement Detail

The depth behind the [overview](../README.md) — how the pilot would actually get scoped, deployed, measured, and hardened for production.

## Contents

1. [Discovery Checklist](#discovery-checklist)
2. [6-Week Engagement Timeline](#6-week-engagement-timeline)
3. [Phased Ramp Schedule](#phased-ramp-schedule)
4. [Change Management for Human Agents](#change-management-for-human-agents)
5. [Evaluation Framework](#evaluation-framework)
6. [Risks & Mitigation](#risks--mitigation)
7. [Production Considerations](#production-considerations)

---

## Discovery Checklist

Week 1 isn't about building — it's about validating that the assumptions in the brief survive contact with reality. Five things to nail down before any prompt gets written:

1. **Baseline & call volumes by intent** — pulled from Genesys disposition codes and call transcripts
2. **IVR structure & routing logic** — current-state documentation
3. **Salesforce data quality & access** — are claim records complete? are fields consistent?
4. **Compliance requirements** — recording consent approach, AI disclosure language
5. **Definition of "success"** — what does the exec sponsor actually care about beyond the 50% containment number?

### Stakeholders to align at kickoff

| Stakeholder | Owns |
|-------------|------|
| Executive Sponsor | Success criteria, go/no-go approval |
| IT / Salesforce Admin | Genesys integration, Salesforce API access, security review |
| Process Owner / Contact Center Ops Lead | Call routing, workforce impact, agent communication & handoffs |
| Compliance / Legal | Recording consent, AI disclosure requirements, scope boundaries |

---

## 6-Week Engagement Timeline

| Week | Phase | Focus |
|------|-------|-------|
| 1 | **Discovery & Design** | Stakeholder alignment, baseline metrics, Salesforce data validation, compliance requirements, conversation flow design |
| 2 | **Design & Integration** | Finalize agent UX, begin Genesys + Salesforce integration |
| 3 | **Build Prototype** | Configure Voice AI agent, build intents, set up guardrails |
| 4 | **Testing & Tuning** | Internal testing, pre-deployment simulation with historical transcripts |
| 5 | **Go-Live Readiness** | Customer UAT, compliance sign-off, human agent change management, go/no-go gate |
| 6+ | **Phased Ramp** | 10% → 25% → 50% → 75% → 100% over 4 weeks, with daily monitoring |

---

## Phased Ramp Schedule

| Timeframe | Traffic % | Focus |
|-----------|-----------|-------|
| Week 1 | 10% | Validate audio quality, catch critical failures, confirm integration success |
| Week 2 | 25% → 50% | Monitor containment rate, accelerate ramp if it holds |
| Week 3 | 75% | Measure KPIs against baseline, continue tuning |
| Week 4+ | 100% | Full deployment, discuss expansion |

### What we monitor during ramp

Containment rate · escalation patterns by intent · failure modes · caller sentiment · **false containment rate** (calls AI "resolved" that result in callbacks — the most dangerous metric to ignore).

---

## Change Management for Human Agents

The agents whose work is being automated need to be brought along, not blindsided.

| Reality | Message to agents |
|---------|-------------------|
| Fewer routine calls (claim status, address updates) | "Repetitive work shifts to AI" |
| More complex work (disputes, escalations, coverage) | "You handle what actually requires judgment" |
| Warm transfers with full context | "Calls arrive with identity verified and intent captured" |

**Rollout playbook:**

1. **Brief** — align the team on what AI handles vs. what escalates to them
2. **Train** — how to receive AI warm transfers with pre-loaded context
3. **Communicate** — "AI handles the repetitive work so you can focus on what matters"
4. **Feedback loop** — agents flag bad transfers during hypercare, fed back into prompt tuning

---

## Evaluation Framework

How to measure whether the agent is actually working:

| Metric | Target | What it tells you |
|--------|--------|-------------------|
| **Containment Rate** | 50%+ within 30 days | % of calls resolved without human transfer. Primary success metric. |
| **Intent Recognition Accuracy** | 95%+ | Did the agent correctly identify what the caller wanted? |
| **Escalation Rate by Intent** | Varies | Where humans are still needed. Informs Phase 2 scope. |
| **AHT (overall)** | < 6.5 min baseline | Is the AI faster than the current human average? |
| **AHT for Escalated Calls** | < baseline | Does pre-verification + context passing reduce human handle time on the calls that *do* escalate? |
| **CSAT** | ≥ baseline | Caller satisfaction shouldn't drop. If it does, the agent needs tuning. |
| **False Containment Rate** | < 5% | Calls AI "resolved" that the caller had to call back about. The most dangerous metric to ignore. |

---

## Risks & Mitigation

| Risk | Mitigation |
|------|------------|
| **IT delays on Genesys / Salesforce access** | Secure access commitments at kickoff — this is dependency #1 |
| **Bad Salesforce data quality** | Validate data quality in Week 1 before building anything |
| **Compliance bottleneck** | Involve Legal in Week 1 discovery, not Week 5 sign-off |
| **Low Week 1 containment** | Set expectations: ramp from 10% → 100%. Containment improves with tuning. |

---

## Production Considerations

What changes moving from prototype to production:

- **Code Tools → middleware API:** Replace mock data with authenticated calls to Salesforce REST API and Genesys Cloud Platform API via a serverless middleware layer
- **Authentication:** OAuth 2.0 for Salesforce, Genesys OAuth for transfers
- **PII governance:** Middleware scopes returned fields — the LLM never sees SSNs, full account numbers, or other sensitive data it doesn't need
- **Audit logging:** Every tool call logged with call ID, timestamp, and data accessed for compliance
- **Call recording consent:** Managed at the Genesys Architect flow level before the call reaches the AI agent
- **Telephony:** Vapi connected to Genesys Cloud via SIP trunk for production call routing
