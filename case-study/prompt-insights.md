# Prompt Insights

Things I learned from actually writing the system prompt for a voice agent that handles real-feeling calls. None of this is in the Vapi docs.

---

## 1. Warm voice ≠ right voice

The default instinct for any AI assistant is "friendly, warm, helpful." That's the wrong default for insurance.

People don't call their insurance company because they're having a good day. They call because they got rear-ended, their basement flooded, their roof caved in, or someone died. A chirpy "Thanks so much for calling EverSure!! How can I make your day amazing?!" is not just off-key — it's actively alienating. *Why are you so happy that I got in an accident?*

What I went with instead: **professional, warm, concise.** Not chipper. Not performatively empathetic either ("Oh no, I'm SO sorry to hear that, that must be so hard!" is its own flavor of fake). The agent is calm, matter-of-fact, takes the caller seriously, and moves quickly to actually helping. If the caller is upset, acknowledge it once and move on:

> "I understand that's frustrating, and I want to help."

Then go solve the problem. Don't dwell.

---

## 2. Section-based, structured prompts

I structured the prompt as discrete sections — `[Identity]`, `[Style]`, `[Response Guidelines]`, `[Identity Verification]`, `[Intent: Claim Status]`, etc. — instead of writing it as one flowing paragraph.

Why this matters:
- **Auditability** — when the agent does something weird, I can point to exactly which section is governing that behavior.
- **Maintainability** — a non-prompt-engineer (an ops lead, a compliance reviewer) can read it and understand which part to touch.
- **Containment** — adding a new intent doesn't risk breaking existing intents because they're scoped to their own block.

---

## 3. `<wait for user response>` is the unsung hero

Without explicit wait markers, the LLM has no concept of conversational turn-taking. It will happily generate the agent's *next three turns* in a single response, steamrolling the caller.

Every multi-step flow in my prompt has `<wait for user response>` after each agent utterance:

```
1. Say: "I'd be happy to help with that. For security purposes, could I get your policy number?"
<wait for user response>
2. Say: "Thank you. And could I get the last name on the account?"
<wait for user response>
3. Call customer_lookup with the policy number and last name.
```

---

## 4. Voice-native formatting changes the output dramatically

I had to override the LLM's default tendency to output text *as written* and force it to output text *as spoken*:

| What the LLM wants to say | What it should say |
|---------------------------|---------------------|
| "Your claim was filed on 3/8/2024" | "Your claim was filed on March eighth, twenty twenty-four" |
| "Your balance is $142.50" | "Your balance is one hundred forty-two dollars and fifty cents" |
| "Call EVS-555-1212" | "Call E-V-S, five-five-five, one-two-one-two" |

ElevenLabs *can* read "$142.50" correctly most of the time, but you don't want "most of the time." Spelling it out at the prompt layer makes the output deterministic. Small thing. Big QA win.

---

## 5. Hide every seam

The caller should never feel like they're talking to a bot using tools. Hard rules in the prompt:

- Never say "function," "tool," "API," "system," or any technical/internal term
- Never mention tool names like `customer_lookup` or `claim_status`
- Don't say "let me query our database" — say "let me pull that up"
- Don't say "I'll route this to the appropriate queue" — say "let me connect you with a billing specialist"

This is one of the easiest places for a voice agent to feel inhuman. The LLM will absolutely default to enterprise-software speak if you don't explicitly forbid it.

---

## 6. The compliance work is invisible but load-bearing

A lot of what looks like product design in this prompt is actually compliance. None of it is in the Vapi docs.

### The first message is hardcoded for a reason

The opener contains, in a specific order:

1. **Company identification** — "Thank you for calling EverSure Insurance"
2. **AI disclosure** — "My name is Alex, and I'm an AI-powered assistant" (required by a growing list of state laws — CA, CO, others moving the same direction)
3. **Recording consent** — "Please be aware that this call may be recorded for quality and training purposes" (mandatory in 2-party-consent states like CA, FL, MA, PA)
4. **Open prompt** — "How can I help you?"

You can't let the LLM improvise this. Every word is doing legal work, and the order matters (consent has to come *before* the conversation, not after the caller has already started talking).

### Coverage / eligibility is a no-go zone for the AI

Not a technical limitation — a regulatory one. Providing coverage determinations typically requires a licensed insurance agent in the relevant state, which means UPL (unauthorized practice) risk if an AI does it, plus E&O liability if the answer is wrong. The prompt has a hard rule:

> NEVER provide specific coverage determinations. NEVER say whether something is or isn't covered.

The agent's job on these calls is verify identity, capture the question, and warm-transfer.

### Data handling rules are the third pillar

What the agent is *forbidden* from doing matters as much as what it does:

- Never read back full policy numbers or account numbers (last four only)
- Never read back SSNs, dates of birth, or payment card numbers
- If the caller volunteers an SSN unprompted, do *not* repeat it back
- Don't share one caller's information with another caller
- If a tool returns weird/empty/conflicting data, don't guess — apologize and escalate

Every line maps to a way the agent could embarrass the company or trigger a compliance incident.

### The pattern

These rules look like they're constraining what the agent can do. They're really the rules that let it ship at all. Every one of them came from a conversation with legal, compliance, or ops — not from a tech spec. The methodology section of the README is partly about this: phase 1 (talk to people) isn't fluff. It's where the load-bearing constraints come from.

---

## 7. One chance to clarify, then escalate

A loop where the agent keeps asking "I'm sorry, could you say that again?" is the worst experience you can build. The caller goes from "this might be useful" to "this is a waste of my time" in under 30 seconds.

My rule: agent gets *one* clarification attempt. If it still doesn't understand, escalate to a human. This is a deliberate choice to under-use the AI. The math works out — a 30-second clarification + transfer is way cheaper than 90 seconds of confused looping followed by an angry caller hanging up.