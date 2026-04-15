# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Vapi voice agent prototype for **EverSure Insurance** — an inbound customer service AI named "Alex" that handles insurance calls. This is a demo/prototype, not a production application. There is no build system, test suite, or runnable code — the repo is a collection of configuration files that define the Vapi assistant.

## Testing the Agent

Test via the [Vapi Dashboard](https://dashboard.vapi.ai) by selecting the **Observe: EverSure Insurance** assistant and starting a test call. The mock customer is **Sarah Mitchell**, policy number **EVS-2024-1847362**.

## Key Files

- `PROMPT.md` — Source of truth for the agent's system prompt. The first section (before the `---`) is the `firstMessage`; everything after is the system message. Changes here must be synced to `assistant.json`'s `model.messages[0].content` and `firstMessage` fields.
- `assistant.json` — Vapi assistant configuration. Contains the model config (GPT-4o via OpenAI), voice config (ElevenLabs), transcriber config (Deepgram), analysis plan, and tool IDs. IDs are redacted with placeholders.
- `tools/*.json` — Vapi Code Tools (TypeScript functions that run on Vapi's infra). Each returns hardcoded mock data. In production these become real API calls to Salesforce, Genesys Cloud, etc.
- `analysis/analysis-plan.json` — Post-call analysis config: summary generation and pass/fail success evaluation.
- `analysis/structured-outputs.json` — Schemas for structured data extracted from each call (intent, containment, sentiment, escalation details, etc.).

## Architecture

All tools are Vapi "code" type tools — inline TypeScript in the JSON, not external servers. They return static mock data for the demo. The five tools are:

| Tool | Purpose | Mock Behavior |
|------|---------|---------------|
| `customer_lookup` | Identity verification (policy # + last name) | Always returns verified for Sarah Mitchell |
| `claim_status` | Claim lookup by number | Returns a fixed "Under Review" water damage claim |
| `payment_info` | Billing/payment details by policy | Returns current account, $142.50/mo auto-pay |
| `update_address` | Update mailing address | Always succeeds, returns old/new address |
| `transfer_to_agent` | Warm transfer to live agent queue | Returns transfer_initiated with context passed |

## Intent Design

Four supported intents with different containment strategies:

- **Claim Status** — Full containment (data lookup, read back)
- **Payment Questions** — Containment for info queries; escalate disputes/refunds to "Billing Specialists"
- **Coverage/Eligibility** — Always escalate to "Licensed Agents" (E&O liability)
- **Address Update** — Full containment (write-back with confirmation)

Anything outside these four intents escalates to "General Support".

## Important Conventions

- `PROMPT.md` and `assistant.json` must stay in sync — the system prompt exists in both places
- `<wait for user response>` markers in the prompt control turn-taking
- The agent verifies identity exactly once per call before sharing any account data
- Coverage/eligibility questions are NEVER answered by the AI — always escalated
- All escalations are warm transfers with reason + conversation summary
- Mock data is hardcoded in tool `code` fields — no external dependencies or environment variables needed for the tools themselves
- `.env` contains `VAPI_TOKEN` for API access
