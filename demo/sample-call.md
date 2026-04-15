# Sample Call Transcripts

Three real scenarios showing how the agent handles its full range — happy path, escalation, and failed verification. All transcripts use the prototype's mock customer:

| Field | Value |
|-------|-------|
| Caller | Sarah Mitchell |
| Policy | EVS-2024-1847362 |
| Claim | CLM-2024-09823 |

---

## 1. Happy Path — Claim Status ✅

> Caller wants to check the status of an open claim. Identity verified, claim looked up, status read back, call ends. **No human involved.**

| Outcome | Handle Time | Queue |
|---------|-------------|-------|
| ✅ Contained by AI | ~90 sec (vs. 6.5 min baseline) | — |

---

**Alex (AI):** Thank you for calling EverSure Insurance. My name is Alex, and I'm an AI-powered assistant here to help you today. Please be aware that this call may be recorded for quality and training purposes. How can I help you?

**Sarah:** Hi, I just wanted to check on the status of my claim.

**Alex:** I'd be happy to help with that. For security purposes, could I get your policy number?

**Sarah:** Yeah, it's E-V-S dash twenty twenty-four dash one eight four seven three six two.

**Alex:** Thank you. And could I get the last name on the account?

**Sarah:** Mitchell.

**Alex:** I've pulled up your account, Sarah. What's the claim number you'd like me to look up?

**Sarah:** It's C-L-M dash twenty twenty-four dash zero nine eight two three.

**Alex:** Your claim for water damage in the kitchen is currently under review. It was last updated on March first, and the next step is an adjuster site visit scheduled for March eighth. Estimated completion is around March fifteenth. Is there anything else I can help you with?

**Sarah:** No, that's everything, thanks.

**Alex:** Thank you for calling EverSure Insurance, Sarah. Have a wonderful day!

---

## 2. Coverage / Eligibility — Always Escalate ⚠️

> Caller asks a coverage question. Agent never answers — it captures the specific question, then warm-transfers to a Licensed Agent. **The licensed agent picks up with full context — no re-verification, no re-explaining.**

| Outcome | AHT Savings (human side) | Queue |
|---------|--------------------------|-------|
| ⚠️ Escalated with full context | ~2-3 min vs. starting from scratch | Licensed Agents |

---

**Alex:** Thank you for calling EverSure Insurance. My name is Alex, and I'm an AI-powered assistant here to help you today. Please be aware that this call may be recorded for quality and training purposes. How can I help you?

**Sarah:** Hi, I'm wondering if my policy covers water damage from a burst pipe.

**Alex:** That's a great question. Just so I make sure I capture it correctly — could you tell me a bit more about what you'd like to know?

**Sarah:** Yeah, a pipe burst in my upstairs bathroom and there's damage to the ceiling below. I want to know if that's covered.

**Alex:** I'd be happy to help with that. For security purposes, could I get your policy number?

**Sarah:** E-V-S dash twenty twenty-four dash one eight four seven three six two.

**Alex:** Thank you. And could I get the last name on the account?

**Sarah:** Mitchell.

**Alex:** I've pulled up your account, Sarah. So you'd like to know if water damage from a burst pipe in your upstairs bathroom — including damage to the ceiling below — is covered by your policy. Is that right?

**Sarah:** Yes, exactly.

**Alex:** Coverage questions like this require review by a licensed agent to make sure you get accurate information. Let me connect you with one of our specialists. I'm connecting you now to a licensed agent. They'll have the details of what we discussed. Thank you for calling EverSure.

---

## 3. Failed Verification — Graceful Escalation 🚫

> Caller can't be verified after two attempts. Agent doesn't loop or shame — it transfers to General Support, where a human can use additional verification methods.

| Outcome | Caller Experience | Queue |
|---------|-------------------|-------|
| 🚫 Verification failed → escalated | No dead-end, no humiliating loop | General Support |

---

**Alex:** Thank you for calling EverSure Insurance. My name is Alex, and I'm an AI-powered assistant here to help you today. Please be aware that this call may be recorded for quality and training purposes. How can I help you?

**Sarah:** I need to update my address.

**Alex:** Sure thing. For security purposes, could I get your policy number?

**Sarah:** I think it's E-V-S dash twenty twenty-three dash one two three four five six seven.

**Alex:** Thank you. And could I get the last name on the account?

**Sarah:** Mitchell.

**Alex:** I wasn't able to verify that information. Could you double-check your policy number and last name?

**Sarah:** Hmm, let me look... I don't have it in front of me.

**Alex:** I'm not able to verify the account with that information. Let me connect you with a team member who can help. I'm connecting you now. They'll have the full details of our conversation so you won't need to repeat anything. Thank you for calling EverSure.

---

## What These Show

| Design Principle | How It Shows Up |
|------------------|-----------------|
| **Verify once per call** | Identity is never re-prompted across the conversation |
| **Voice-native readback** | Dates and numbers spelled naturally ("March eighth" not "3/8") |
| **System internals invisible** | Caller never hears "I'm calling the API" or tool names |
| **Escalation with context** | Every transfer includes intent, identity status, and conversation summary |
| **No dead ends** | One clarification, then route to a human — never trap the caller in a loop |
