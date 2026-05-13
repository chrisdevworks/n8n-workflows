# AI Client Enquiry Processor — Part 1

**Assessment:** AI Developer Pre-Interview Assessment  
**Position:** AI Developer at Strata Management Consultants  
**Points:** 40  
**Tool:** n8n (self-hosted or cloud)

---

## What It Does

This n8n workflow automates the intake and triage of client enquiries for a strata management company. When a client enquiry arrives, the workflow:

1. Validates the input (presence, length bounds, required fields)
2. Sends the enquiry to Claude (claude-3-5-sonnet) via the Anthropic API
3. Returns a structured JSON result for staff containing:
   - **Classification** — what type of enquiry it is (8 categories)
   - **Confidence score** — how certain the AI is (0–100%)
   - **Urgency level** — low / medium / high
   - **One-line summary** — for quick scanning
   - **Key details** — extracted bullet points
   - **Suggested email response** — ready-to-send draft
   - **Recommended action** — concrete next step for staff
   - **Routing suggestion** — which team should handle it

---

## Setup

### Prerequisites
- n8n instance (self-hosted or [n8n.io cloud](https://n8n.io))
- Anthropic API key ([console.anthropic.com](https://console.anthropic.com))

### Steps

1. **Import the workflow**
   - In n8n → Workflows → Import from File
   - Select `part1-ai-enquiry-processor.json`

2. **Create the API credential**
   - Go to **Credentials → New → Header Auth**
   - Name it exactly: `Anthropic API Key`
   - Header Name: `x-api-key`
   - Header Value: your `sk-ant-...` key

3. **Link the credential**
   - Open the **"Call Claude API"** node
   - Under Authentication, select `Anthropic API Key`

4. **Activate the workflow**
   - Toggle the workflow to Active
   - Copy the webhook URL shown in the "Receive Enquiry" node

---

## API Usage

### Request

```
POST <your-n8n-base-url>/webhook/process-enquiry
Content-Type: application/json
```

```json
{
  "enquiry": "Hi, I received my quarterly levy notice and the amount seems higher than last quarter. Can you explain what the increase covers?",
  "sender_name": "Jane Smith",
  "sender_email": "jane.smith@example.com",
  "sender_phone": "+61 400 000 000"
}
```

Only `enquiry` is required. `sender_name`, `sender_email`, and `sender_phone` are optional.

### Successful Response (HTTP 200)

```json
{
  "success": true,
  "http_status": 200,
  "enquiry": {
    "original_text": "Hi, I received my quarterly levy notice...",
    "sender_name": "Jane Smith",
    "sender_email": "jane.smith@example.com",
    "sender_phone": "+61 400 000 000",
    "received_at": "2026-05-13T09:30:00.000Z"
  },
  "analysis": {
    "classification": "billing_enquiry",
    "confidence_score": 0.96,
    "confidence_percent": "96%",
    "urgency": "medium",
    "summary": "Client querying a levy increase on their quarterly notice",
    "key_details": [
      "Quarterly levy notice received",
      "Amount higher than previous quarter",
      "Client requesting explanation of the increase"
    ]
  },
  "staff_guidance": {
    "recommended_action": "Forward to Accounts Team with levy schedule for this lot",
    "routing_suggestion": "Accounts Team",
    "suggested_response": "Dear Jane,\n\nThank you for reaching out regarding your quarterly levy notice.\n\nLevy amounts can vary each quarter based on the approved budget set by the owners corporation, which covers building insurance, maintenance funds, and administration costs. An increase may also reflect special levies raised for specific capital works.\n\nI will review the levy schedule for your lot and send through a detailed breakdown shortly. If you have any further questions in the meantime, please don't hesitate to ask.\n\nKind regards,\n[Staff Name]\nStrata Management Consultants"
  },
  "meta": {
    "processed_at": "2026-05-13T09:30:01.243Z",
    "model": "claude-3-5-sonnet-20241022",
    "tokens_used": { "input": 612, "output": 287 },
    "workflow_version": "1.0.0"
  }
}
```

### Validation Error (HTTP 400)

```json
{
  "success": false,
  "http_status": 400,
  "error": "missing_enquiry",
  "message": "No enquiry text provided. Include an \"enquiry\" field in your request body."
}
```

---

## Design Decisions

### Why n8n?

n8n provides a visual, auditable workflow that non-technical staff can understand and maintain. Each processing step is a discrete, inspectable node — making it easy to debug, modify, or hand off to another developer.

### Model Choice — Claude 3.5 Sonnet

`claude-3-5-sonnet-20241022` was chosen for its balance of accuracy and speed. Classification tasks with structured JSON output benefit from a capable model; Sonnet handles the structured-output constraint reliably without needing function calling or JSON mode.

### Prompt Engineering

The system prompt does three things:
- **Constrains output format** — specifies an exact JSON schema so the Code node can parse it reliably
- **Defines all 8 classification types** with plain-English descriptions to reduce ambiguity
- **Provides persona and context** — "strata management company, Australia" grounds the model's vocabulary for domain-specific terms (levies, by-laws, NCAT, owners corporation)

The `suggested_response` uses `[Staff Name]` as an explicit placeholder so staff know to personalise it before sending — avoiding accidental dispatch of template text.

### Error Handling

Three error categories are handled:

| Error Type | Cause | HTTP Status |
|---|---|---|
| Validation error | Missing/short/long enquiry | 400 |
| Anthropic API error | Bad key, rate limit, network | 500 (in `success: false` body) |
| Parse error | AI returned non-JSON | 500 (in `success: false` body) |

The `neverError: true` option on the HTTP Request node prevents n8n from halting execution on non-2xx responses, so all Anthropic errors flow into the Parse node and are returned gracefully to the caller.

A regex fallback (`rawText.match(/\{[\s\S]*\}/)`) strips accidental markdown code fences from the AI response before JSON parsing — a common edge case when models ignore the "no markdown" instruction.

### Confidence Scoring (Bonus)

The prompt asks the model to rate its own certainty (0.0–1.0). The result is returned as both `confidence_score` (float) and `confidence_percent` (string) so the front-end can use whichever format suits. Low-confidence results (< 0.6) signal to staff that manual review is warranted.

### Scaling to a Larger Workflow

This webhook-based design plugs directly into:
- **Email systems** — e.g. Gmail trigger node replaces the webhook; staff never need to paste text manually
- **CRM integration** — the classification and routing outputs map to standard CRM fields (ticket type, priority, assignee)
- **Task queue** — high-urgency results can fan out to a Slack alert or auto-create a task in Asana/Monday
- **Audit log** — add a Google Sheets or database node after "Parse & Format Result" to log every enquiry, its classification, and the confidence score for reporting

---

## Workflow Diagram

```
[POST /webhook/process-enquiry]
         │
         ▼
  [Validate Input]
    • check field exists
    • length 10–5000 chars
         │
    ┌────┴─────┐
   valid?    invalid
    │           │
    ▼           ▼
[Build AI     [Format Validation Error]
 Request]          │
    │              ▼
    ▼        [Respond 400]
[Call Claude API]
    │
    ▼
[Parse & Format Result]
  • handle API errors
  • handle parse errors
  • normalise all fields
    │
    ▼
[Respond 200 — JSON]
```

---

## Tools Used

| Tool | Purpose |
|---|---|
| **n8n** | Workflow orchestration |
| **Anthropic Claude API** | Enquiry classification + response generation |
| **Webhook node** | HTTP entry point |
| **Code nodes (JS)** | Validation, prompt construction, response parsing |
| **HTTP Request node** | Anthropic API call |
| **Respond to Webhook** | Returns JSON to caller |
