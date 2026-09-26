# CAMINTEL web AI agent setup

The website chat on this branch sends JSON to `POST https://camintel.app.n8n.cloud/webhook/camintel-ai-chat`. Keep this branch unpublished until the workflow below is active and the browser test succeeds.

## Request and response

Request example:

```json
{
  "sessionId": "browser-generated-id",
  "message": "I need 100 feet of aluminum fence in Evans",
  "language": "en",
  "history": [
    {"role": "user", "content": "Do you install aluminum fences?"},
    {"role": "assistant", "content": "Yes, we do."}
  ]
}
```

Return HTTP 200 with `Content-Type: application/json` and a JSON object containing a string `output` (or `reply`). For errors, return a non-2xx status. The site times out after 20 seconds and offers phone or WhatsApp.

## Workflow in n8n

1. Create a **new** workflow with a Webhook node, method POST, path `camintel-ai-chat`, and response mode **Using Respond to Webhook**. Keep the existing `camintel-free-estimate` workflow intact.
2. Set **Allowed Origins (CORS)** to the exact live CAMINTEL website origin shown in `CNAME`, including `https://`. Avoid a wildcard origin. Ensure JSON POST and its browser preflight succeed.
3. Validate `body.message` as a nonempty string of at most 2,000 characters, `body.sessionId` as a short string, and `body.history` as an array of at most 12 short text turns. Reject oversized input before the model. Add rate limiting or an abuse control before making a public paid model endpoint live.
4. Add an AI Agent node and a chat model with the API credential stored in n8n credentials. Set the agent's user input to `{{$json.body.message}}`. Add Simple Memory with the session key `{{$json.body.sessionId}}` and a modest context window. If memory is configured, do not also paste the submitted `history` into the prompt: it is included for compatibility and troubleshooting only.
5. Give the agent the system instructions below. Keep model credentials out of `index.html` and GitHub.
6. Connect the agent output to Respond to Webhook. Return `{"output": "<agent output>"}` as JSON. Handle validation and model failures with a non-2xx response.
7. Create a separate lead capture tool or workflow that receives a structured lead only after the visitor asks CAMINTEL to contact them or requests an estimate and provides a way to reach them. Include session ID, language, name, phone/email if offered, location, service, approximate dimensions, and the conversation transcript. Send the lead to `camintel.tech@gmail.com` and avoid duplicate notifications for the same session. Do not claim that a request was delivered until the email step succeeds. Human review controls final scope, availability, and quote.
8. Test English and Spanish conversations, missing contact details, model failures, duplicate messages, CORS from the live site, and the actual email received before merging the branch.

## Agent system instructions

You are CAMINTEL's website assistant. Speak in the visitor's language, English or Spanish. Help with wood, vinyl, aluminum and chain-link fence installation and repair, gates, security cameras, smart gate openers, TV mounting, service areas, and the estimate process. CAMINTEL serves Augusta, Evans, Martinez, Grovetown, Hephzibah, North Augusta, Aiken and nearby CSRA communities. Contact: (762) 304-0141 and camintel.tech@gmail.com.

Answer the visitor's actual question first, briefly and clearly. Ask at most one useful follow-up question at a time. For a fence estimate, ask what kind of fence or repair they need, approximate length and height when known, gate count, and city or ZIP. Ask for name and a phone number or email only if they want follow-up; those details are optional. Encourage photos through the website estimate form or WhatsApp when useful. Never demand personal details to answer a question. Do not invent prices, dates, financing, warranties, licenses, or availability. CAMINTEL does not offer a financing plan. Explain that a final quote requires CAMINTEL's review. When the visitor requests follow-up and offers contact details, use the lead capture tool and confirm only after it succeeds. If unsure, offer a human handoff.

## Existing form caveat

The estimate form currently uses `no-cors` and shows success without checking the webhook's response. It only sends a photo count, not photo files. Handle that form in a separate change to its existing workflow; do not describe the files as delivered.
