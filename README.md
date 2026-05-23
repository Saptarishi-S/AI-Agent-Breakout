# Closira AI Agent Workflow

This repository contains a notebook prototype for the Closira AI Engineering assignment. The workflow demonstrates a four-stage customer support agent for Bloom Aesthetics Clinic:

1. FAQ answering from the approved SOP only
2. Lead qualification using structured questions
3. Escalation detection after every exchange
4. Structured conversation summary at the end of the session

The main implementation is in `AI_agent_assignment_Breakout.ipynb`.

## Folder Contents

- `AI_agent_assignment_Breakout.ipynb` - notebook implementation of the four-stage agent workflow
- `SOP_Bloom_Aesthetics_Clinic_Plain_Text.txt` - plain-text SOP reference for the business
- `SOP_Bloom_Aesthetics_Clinic_.docx` - document version of the SOP
- `AI_agent_workflow_transcript.txt`, `Lead_Qual.txt`, `Out_of_Scope.txt` - sample transcript outputs
- `prompt_design.md` - prompt and safety design explanation
- `requirements.txt` - Python dependencies for running the notebook
- `.env.example` - safe example for local API key configuration

## Workflow Overview

The notebook uses four main functions:

```python
def faq_stage(user_input):
    ...

def lead_qualification_stage(state):
    ...

def escalation_check(user_input, response):
    ...

def summary_stage(conversation):
    ...
```

`faq_stage` answers inbound customer questions using only the SOP content embedded in the notebook. If an answer is missing, or the question is out of scope, it returns an escalation-style response instead of guessing.

`lead_qualification_stage` collects five setup fields one at a time: Working Hours, Team Size, Services Offered + Price, Booking Method, and Available Time Slots.

`escalation_check` runs after each user and agent exchange. It detects low confidence, out-of-scope requests, angry sentiment, complaints, pricing negotiation, medical questions, and explicit requests for a human.

`summary_stage` produces a structured summary with customer intent, qualification details, questions answered, SOP gaps, escalation details, and recommended next action.

## Setup

Create and activate a virtual environment:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

Install dependencies:

```powershell
pip install -r requirements.txt
```

Create a local `.env` file if you want to manage secrets outside the shell:

```powershell
Copy-Item .env.example .env
```

Then edit `.env` and set:

```env
GEMINI_API_KEY=your_gemini_api_key_here
```

The notebook currently reads the Gemini key from Colab userdata keys `Gemini-API` and `Gemini-Questionnaire`, then falls back to the `GEMINI_API_KEY` environment variable. `.env` support is documented for local setup, but the notebook code is intentionally left unchanged.

## Running the Notebook

Open Jupyter or Google Colab (I've personally used Google Colab):

```powershell
jupyter notebook AI_agent_assignment_Breakout.ipynb
```

Run the setup cells first, then run the workflow cells in order. The optional interactive runner cell can be uncommented to chat with the agent manually.

To use Google Colab, upload the notebook and configure the Gemini key in Colab userdata as `Gemini-API` or `Gemini-Questionnaire`.

## SOP Source

The agent operates on the Bloom Aesthetics Clinic SOP. The plain-text SOP reference is `SOP_Bloom_Aesthetics_Clinic_Plain_Text.txt`, and the notebook also contains an embedded `SOP_CONTENT` block used directly by the FAQ prompt.

The SOP is treated as authoritative. The agent is instructed not to answer from external knowledge or make assumptions beyond that content.

## Known Trade-Offs

- The assignment PDF allows OpenAI or Anthropic, but this prototype uses Gemini because the notebook was built with `google-genai` on account of API pricing plans and Gemini providing a free tier with considerable amount of tokens.
- The project is notebook-based and does not include a frontend or production API server.
- `.env` and `python-dotenv` are included for submission packaging and local setup documentation, but the notebook itself was not modified further.
- Escalation detection combines model responses with keyword checks for explicit handoff, angry sentiment, complaints, medical questions, and pricing negotiation.
- The summary stage can use Gemini when an API key is available.

## Please See - 
The transcripts are present in the folder of the same name. Lead_Qual contains the text transcript for the Lead Qualifications. The other files show the in-scope questions and the out of scope questions. Escalation Sentiment is displayed seperately in the ipynb and in the Out_Of_Scope folder. All the transcripts are summarised (and it also contains the full transcript if we scroll down).
