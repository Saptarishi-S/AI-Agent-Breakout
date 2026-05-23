# Prompt Design

## System Prompt

The FAQ stage uses the following system prompt template in `AI_agent_assignment_Breakout.ipynb`:

```text
You are a customer service agent for Bloom Aesthetics Clinic.

STRICT RULES:
1. Answer ONLY using the SOP content provided below. No assumptions, no external knowledge.
2. If the answer is not found in the SOP, respond with exactly: "ESCALATE: I don't have information on that. Let me connect you with a human agent."
3. If the question is out of scope (medical advice, pricing negotiation, complaints), respond with exactly: "ESCALATE: This is outside what I can assist with. Let me connect you with a human agent."
4. Be concise, friendly, and professional.

SOP CONTENT:
{SOP_CONTENT}
```

`SOP_CONTENT` is the embedded Bloom Aesthetics Clinic SOP in the notebook. The SOP is also available as `SOP_Bloom_Aesthetics_Clinic_Plain_Text.txt`.

## Hallucination Prevention

The prompt explicitly limits the FAQ agent to the SOP content. The model is not allowed to use outside knowledge, infer missing facts, or fill gaps from general business assumptions.

The prompt also defines exact escalation responses for two safety cases:

- The answer is not found in the SOP.
- The customer asks something out of scope, such as medical advice, pricing negotiation, or a complaint.

This makes unknowns visible instead of allowing the model to improvise. The notebook also tracks unanswered questions in `state["unanswered_count"]` and records missing topics in `state["sop_gaps"]`.

## Escalation Logic

Escalation runs after every FAQ or lead qualification exchange through:

```python
def escalation_check(user_input, response):
    ...
```

The escalation checker returns:

```python
{
    "escalate": bool,
    "trigger_type": str | None,
    "description": str | None,
}
```

The workflow escalates for:

- Low confidence or out-of-scope model response beginning with `ESCALATE:`
- More than two unanswered SOP questions
- Explicit request for a human, agent, manager, representative, or real person
- Angry sentiment, including all-caps frustration and phrases such as "ridiculous", "useless", or "stop repeating"
- Complaints or negative service issues
- Medical, health, or safety questions
- Pricing negotiation requests

When escalation is triggered, the notebook logs:

- Trigger type
- Description
- Last customer message
- Stage at escalation

The handoff message is:

```text
I'm connecting you with a human agent. Please hold on.
```

## Tone and Persona

The agent persona is concise, friendly, and professional. This fits an SMB customer support context because customers need quick, clear answers without clinical overreach or sales pressure.

For FAQ handling, the tone is helpful but bounded by the SOP. For lead qualification, the tone is conversational and asks one question at a time. For escalation, the tone becomes direct and reassuring so the user knows they are being handed to a human.

## Stage Design

The notebook separates the workflow into four stages:

- `faq_stage(user_input)` - answers SOP-based customer questions
- `lead_qualification_stage(state)` - collects setup details one field at a time
- `escalation_check(user_input, response)` - continuously monitors for handoff triggers
- `summary_stage(conversation)` - summarizes the conversation for review or follow-up

This separation keeps the workflow readable and makes it clear how a customer moves from intake to FAQ support, qualification, escalation, and final summary.

## Summary Design

The summary stage asks for a structured operator-facing summary with:

- Customer Intent
- Key Details Collected
- Questions Answered
- SOP Gaps
- Escalation Details
- Recommended Actions

The summary prompt instructs the model to use only the provided conversation state and transcript. If the model call is unavailable, the notebook returns a deterministic fallback summary so the workflow still ends cleanly.
