# Postbox - Full API Reference

> Postbox is an agent-native structured data collection platform. Forms are the base unit - a well-defined collection of fields that expresses intent. A contact form can be name, email, message. A bug report form can be title, reproduction steps. Create a form by defining the schema, get an endpoint, and start receiving validated submissions from HTML forms, scripts, or AI agents. No backend to build, no database to manage, no validation to write.

Base URL: `https://usepostbox.com`

Two interfaces:

- **REST API** (Bearer token auth, all plans) - Create forms, collect submissions, configure processing pipelines, and integrate with your existing systems via webhooks and connectors.
- **MCP Server** (OAuth 2.1, Pro plan) - Connect AI assistants directly to your data. Build custom workflows, explore submissions conversationally, spot patterns, and draft replies from any MCP client.

## Authentication

All API requests require a Bearer token:

```
Authorization: Bearer {api_key}
```

Generate API keys at https://usepostbox.com/integrations/api-keys

Convention: store as `POSTBOX_API_TOKEN` environment variable.

API keys are long-lived and revocable. Each key has a name for identification.

Unauthenticated requests return `401 Unauthorized` (plain text).

All authenticated API responses include credit usage headers:

| Header | Description |
|--------|-------------|
| `X-Postbox-Credits-Remaining` | Remaining AI credits for the current period |
| `X-Postbox-Metered` | Pro users: `"true"` if monthly credits are depleted and usage is metered, `"false"` otherwise. Free users: always `"false"` (no metered billing - AI features stop when credits run out) |

## Quick Start

1. Create a form:

```
POST /api/forms
Authorization: Bearer {api_key}
Content-Type: application/json

{
  "name": "Contact",
  "slug": "contact",
  "visibility": "public",
  "fields_schema": {
    "fields": [
      { "name": "name", "type": "string", "required": true },
      { "name": "email", "type": "email", "required": true },
      { "name": "message", "type": "string", "required": false }
    ]
  }
}
```

Response `201 Created`:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Contact",
  "slug": "contact",
  "visibility": "public",
  "submission_token": null,
  "created_at": "2026-03-01T10:00:00Z",
  "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/contact"
}
```

2. Submit data to the endpoint (no auth required for public forms):

```bash
curl -X POST https://usepostbox.com/api/{opaque_segment}/f/contact \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com", "message": "Hello"}'
```

Two API calls. From nothing to a working endpoint.

## Forms API

All form management endpoints require Bearer token authentication.

### Create Form

```
POST /api/forms
```

Request body:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Human-readable form name (1-100 characters) |
| `slug` | string | yes | URL-safe identifier (1-64 characters). Pattern: `^[a-z0-9]+(?:-[a-z0-9]+)*$`. Unique per user. |
| `visibility` | string | yes | `"public"` - anyone can submit, no auth needed. `"private"` - submissions require `Authorization: Bearer {submission_token}`. |
| `fields_schema` | object | yes | Defines accepted fields, types, and validation. See Field Schema below. |
| `spam_protection_enabled` | boolean | no | Enable spam detection. Default: `false` |
| `spam_protection_strategy` | string | no | `"standard"` - heuristic detection (all plans). `"intelligent"` - AI-powered analysis (uses AI credits). Default: `"standard"` |
| `localisation_enabled` | boolean | no | Auto-detect language and translate submissions. Uses AI credits. Default: `false` |
| `smart_reply_enabled` | boolean | no | Generate AI replies using a knowledge base. Uses AI credits. Default: `false` |
| `smart_replies_mode` | string | no | `"draft"` - save for review. `"auto"` - if the submission contains an email field, Postbox sends the reply directly to the submitter; if no email is present, the reply is drafted and stored for the form owner to use however they choose. Default: `"draft"` |
| `knowledge_base_id` | string | no | UUID of the knowledge base for smart replies. Required when `smart_reply_enabled` is `true`. See Knowledge Bases section. |
| `webhook_enabled` | boolean | no | Enable webhook delivery for new submissions. Default: `false` |
| `webhook_url` | string | no | HTTPS URL to receive webhook payloads. Required when `webhook_enabled` is `true`. |
| `webhook_secret` | string | no | Secret for HMAC-SHA256 webhook signing (8-128 characters). Required when `webhook_enabled` is `true`. |

Response `201 Created`:

```json
{
  "id": "uuid",
  "name": "string",
  "slug": "string",
  "visibility": "public|private",
  "submission_token": "string|null",
  "created_at": "ISO 8601",
  "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/{slug}"
}
```

Important: `submission_token` is only returned for private forms and shown only once. Store it immediately. This token is what submitters (or your backend/agent) must include as a Bearer token when posting to a private form.

The `endpoint` field is the pre-computed submission URL, ready to use.

Response `422 Unprocessable Entity`:

```json
{ "errors": { "slug": ["has already been taken"] } }
```

### List Forms

```
GET /api/forms
```

Response `200`:

```json
{
  "forms": [
    {
      "id": "uuid",
      "name": "string",
      "slug": "string",
      "visibility": "public|private",
      "submission_count": 0,
      "created_at": "ISO 8601"
    }
  ]
}
```

### Get Form

```
GET /api/forms/{id}
```

Response `200`:

```json
{
  "id": "uuid",
  "name": "string",
  "slug": "string",
  "visibility": "public|private",
  "schema": {
    "fields": [{ "name": "email", "type": "email", "required": true }]
  },
  "submission_count": 0,
  "created_at": "ISO 8601"
}
```

Response `404`:

```json
{ "errors": "Form not found" }
```

### Update Form

```
PUT /api/forms/{id}
```

Same fields as Create, all optional. Changes to `fields_schema` auto-create a new versioned endpoint. See Schema Versioning for details.

Response `200`: Same shape as Get Form.

### Delete Form

```
DELETE /api/forms/{id}
```

Response `200`: Returns the deleted form.

Response `404`:

```json
{ "errors": "Form not found" }
```

## Field Schema

The `fields_schema` object defines what data a form accepts.

```json
{
  "fields": [
    {
      "name": "field_name",
      "type": "string|email|number|boolean|date",
      "required": true,
      "honeypot": false
    }
  ]
}
```

### Field Types

| Type | Validation | Example Value |
|------|-----------|---------------|
| `string` | Must be a string | `"Hello world"` |
| `email` | Valid email format (must contain `@`) | `"user@example.com"` |
| `number` | Integer or float | `42` or `3.14` |
| `boolean` | `true` or `false` | `true` |
| `date` | ISO 8601 date string (YYYY-MM-DD) | `"2026-03-01"` |

### Honeypot Fields

Honeypot fields are invisible spam traps. Set `"honeypot": true` on any field to use it as a honeypot. A field cannot be both `required` and `honeypot`.

**How honeypots work:** The field is included in the form schema but should be hidden from human users via CSS. Legitimate users never see or fill these fields. Bots, which typically fill every field they find, will populate the honeypot and trigger the spam filter.

**Best field names for honeypots:** Use names that look like real, attractive fields to bots: `website`, `company`, `url`, `fax`, `phone2`. Avoid names like `honeypot` or `trap` which sophisticated bots may recognize.

**CSS hiding technique:** Do NOT use `display:none` alone. Some bots detect it and skip those fields. Use a more robust approach:

```html
<div style="position:absolute; left:-9999px; top:-9999px; opacity:0; pointer-events:none;">
  <input type="text" name="website" tabindex="-1" autocomplete="off" />
</div>
```

**Example honeypot field in schema:**

```json
{ "name": "website", "type": "string", "required": false, "honeypot": true }
```

## Private Forms

Private forms require authentication to submit data. This is useful for internal tools, backend-to-backend integrations, or any scenario where you don't want arbitrary public submissions.

### Full Flow

1. **Create the form** with `"visibility": "private"`:

```json
{
  "name": "Internal Feedback",
  "slug": "internal-feedback",
  "visibility": "private",
  "fields_schema": {
    "fields": [
      { "name": "employee_id", "type": "string", "required": true },
      { "name": "feedback", "type": "string", "required": true }
    ]
  }
}
```

2. **Store the submission_token** from the response. It is only shown once:

```json
{
  "id": "uuid",
  "submission_token": "pb_sub_a1b2c3d4e5f6...",
  "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/internal-feedback"
}
```

If you lose the `submission_token`, you must delete and recreate the form to get a new one.

3. **Submit with the submission token** as a Bearer token:

```bash
curl -X POST https://usepostbox.com/api/{opaque_segment}/f/internal-feedback \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer pb_sub_a1b2c3d4e5f6..." \
  -d '{"employee_id": "E1234", "feedback": "Great quarter"}'
```

Without the token, the endpoint returns `401 Unauthorized`.

## Submissions

### Endpoint URL Format

Submission endpoints follow this pattern:

```
https://usepostbox.com/api/{opaque_segment}/f/{slug}
```

The `...` represents an opaque, server-generated path segment. Do not construct this URL manually. Instead, use the `endpoint` field returned in the form creation or retrieval response - it contains the complete, ready-to-use submission URL.

### Submit Data

```
POST /api/{opaque_segment}/f/{slug}
```

No authentication required for public forms. Private forms require `Authorization: Bearer {submission_token}`.

Accepts `Content-Type: application/json` only. JSON payloads give you structured validation errors with per-field details, which is not possible with form-urlencoded submissions.

CORS is handled automatically - submit from any origin.

Request body: JSON object with field names matching the form schema.

```json
{
  "name": "Alice",
  "email": "alice@example.com",
  "message": "Hello from a form"
}
```

Response `201 Created`:

```json
{
  "id": "uuid",
  "data": {
    "name": "Alice",
    "email": "alice@example.com",
    "message": "Hello from a form"
  },
  "created_at": "ISO 8601"
}
```

Response `400` (validation error):

```json
{
  "error": {
    "code": "validation_error",
    "message": "Validation failed",
    "details": { "email": ["is required"], "age": ["invalid number"] }
  }
}
```

Response `401` (private form, missing or invalid submission token):

```json
{
  "error": {
    "code": "unauthorized",
    "message": "Unauthorized (missing or invalid API key)"
  }
}
```

Response `404` (form not found):

```json
{
  "error": {
    "code": "form_not_found",
    "message": "Not found"
  }
}
```

Response `429` (submission limit reached on free plan):

```json
{
  "error": {
    "code": "plan_limit_exhausted",
    "message": "Submission limit reached.",
    "upgrade_url": "https://usepostbox.com/settings/billing"
  }
}
```

### Discover Form Schema

```
GET /api/{opaque_segment}/f/{slug}
```

Returns the form's field schema so agents and scripts can discover what to submit.

Response `200`:

```json
{
  "name": "Contact",
  "slug": "contact",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "email", "type": "email", "required": true },
    { "name": "message", "type": "string", "required": false }
  ]
}
```

This endpoint is key for agent-native data collection. See Agent Discovery Pattern for details.

### Preview Form (Browser)

```
GET /f/{opaque_segment}/{slug}
```

Opens an HTML preview of the form in the browser. Only available for public forms. Useful for testing or sharing a quick link to your form.

### List Submissions

```
GET /api/forms/{form_id}/submissions
```

Requires Bearer token authentication. Returns paginated submissions for a form you own, sorted by `created_at` descending (newest first).

Query parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `filter` | string | `"inbox"` | `"inbox"` (non-spam only), `"spam"` (spam only), or `"all"` |
| `page` | integer | 1 | Page number |
| `page_size` | integer | 10 | Results per page |

Response `200`:

```json
{
  "data": [
    {
      "id": "uuid",
      "data": { "name": "Alice", "email": "alice@example.com" },
      "processing_status": "completed",
      "processing_reason": null,
      "spam_flag": false,
      "spam_confidence": null,
      "spam_reason": null,
      "original_language": null,
      "translated_data": null,
      "reply_status": "pending",
      "reply_content": null,
      "reply_reason": null,
      "replied_at": null,
      "created_at": "ISO 8601"
    }
  ],
  "meta": {
    "current_page": 1,
    "total_pages": 5,
    "total_count": 42,
    "page_size": 10
  }
}
```

Response `404`:

```json
{ "error": { "code": 404, "message": "Form not found" } }
```

### Get Submission

```
GET /api/forms/{form_id}/submissions/{id}
```

Requires Bearer token authentication.

Response `200`:

```json
{
  "id": "uuid",
  "data": { "name": "Alice", "email": "alice@example.com" },
  "processing_status": "completed",
  "processing_reason": null,
  "spam_flag": false,
  "spam_confidence": null,
  "spam_reason": null,
  "original_language": null,
  "translated_data": null,
  "reply_status": "pending",
  "reply_content": null,
  "reply_reason": null,
  "replied_at": null,
  "created_at": "ISO 8601"
}
```

Response `404`:

```json
{ "error": { "code": 404, "message": "Submission not found" } }
```

### Delete Submission

```
DELETE /api/forms/{form_id}/submissions/{id}
```

Requires Bearer token authentication.

Response `200`: Returns the deleted submission (same shape as Get Submission).

Response `404`:

```json
{ "error": { "code": 404, "message": "Submission not found" } }
```

### Processing Pipeline

The submission endpoint responds `201 Created` immediately. All processing happens async:

1. **Validation** - Schema-enforced. Required fields, types, constraints.
2. **Spam detection** - If enabled, runs your chosen strategy. Spam is flagged silently (submitter always gets `201`).
3. **Translation** - If enabled, detects language and translates. Original preserved.
4. **Smart replies** - If enabled with a knowledge base, generates and optionally sends a reply.
5. **Notifications** - Webhooks, Discord/Slack connectors, and email notifications are delivered after processing.

If spam is caught, the pipeline stops. Translation and smart replies are skipped.

### Processing Status

Every submission has a `processing_status` field that tracks where it is in the async pipeline:

| Status | Meaning |
|--------|---------|
| `pending` | Submission received, processing not yet started |
| `processing` | Currently being processed (spam check, translation, etc.) |
| `completed` | All pipeline steps finished successfully (including when spam is detected) |
| `skipped` | AI processing skipped (free-tier user with no remaining credits) |
| `failed` | Processing encountered an error |

The `processing_reason` field contains a human-readable explanation when processing does not complete normally:
- When `skipped`: `"ai_credits_exhausted"`
- When `failed`: describes the error (e.g., `"Processing error: ..."`)
- Otherwise: `null`

The webhook fires after processing completes (or fails), so the webhook payload always contains the final `processing_status`.

## Schema Versioning

Every change to `fields_schema` via `PUT /api/forms/{id}` auto-creates a new versioned endpoint. This is how Postbox guarantees that existing integrations never break.

**How it works:**

- When you update a form's `fields_schema`, the form's internal version increments automatically.
- The submission endpoint URL stays the same (`/api/{opaque_segment}/f/{slug}`), but Postbox routes submissions to the correct schema version based on when the integration was set up.
- Old submissions retain their original shape. If version 1 had fields `[name, email]` and version 2 adds `[name, email, phone]`, version 1 submissions still only contain `name` and `email`.
- You don't need to manage versions manually. Just update the schema whenever you need to, and Postbox handles backward compatibility.

**When to use this:** Change your schema freely as your product evolves. Add new fields, remove old ones, change types. Existing frontend forms, scripts, and agents that submit to the old schema will continue working without any changes on their end.

## Knowledge Bases

Knowledge bases are collections of text content that power smart replies. When a submission comes in, Postbox uses the linked knowledge base to generate a contextually relevant reply.

**What goes in a knowledge base:** FAQs, product documentation, support articles, company policies, pricing details, or any text content that would help auto-generate replies to form submissions. Think of it as the "brain" behind smart replies.

### Create Knowledge Base

```
POST /api/knowledge_bases
```

Request body:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | yes | Human-readable name (1-100 characters). Example: "Support FAQ", "Product Docs" |
| `content` | string | yes | The knowledge base text content. This is the material Postbox uses to generate smart replies. Can be plain text, markdown, or structured Q&A. |

Response `201 Created`:

```json
{
  "id": "uuid",
  "name": "string",
  "content": "string",
  "created_at": "ISO 8601"
}
```

**Example:**

```json
{
  "name": "Support FAQ",
  "content": "## Shipping\nWe ship worldwide. Standard shipping takes 5-7 business days. Express shipping takes 1-2 business days.\n\n## Returns\nWe accept returns within 30 days of purchase. Items must be unused and in original packaging.\n\n## Pricing\nAll prices include tax. We offer a 10% discount for orders over $100."
}
```

After creating a knowledge base, link it to a form by setting `knowledge_base_id` when creating or updating the form, and enabling `smart_reply_enabled: true`.

### List Knowledge Bases

```
GET /api/knowledge_bases
```

Response `200`:

```json
{
  "knowledge_bases": [
    {
      "id": "uuid",
      "name": "string",
      "created_at": "ISO 8601"
    }
  ]
}
```

### Get Knowledge Base

```
GET /api/knowledge_bases/{id}
```

Response `200`:

```json
{
  "id": "uuid",
  "name": "string",
  "content": "string",
  "created_at": "ISO 8601"
}
```

Response `404`:

```json
{ "errors": "Knowledge base not found" }
```

### Update Knowledge Base

```
PUT /api/knowledge_bases/{id}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | no | Updated name |
| `content` | string | no | Updated content |

Response `200`: Same shape as Get Knowledge Base.

### Delete Knowledge Base

```
DELETE /api/knowledge_bases/{id}
```

Response `200`: Returns the deleted knowledge base.

**Important:** If a knowledge base is linked to a form with smart replies enabled, deleting it will cause smart reply processing to fail for that form. Update the form to disable smart replies or link a different knowledge base first.

## Smart Replies

Smart replies auto-generate responses to form submissions using a knowledge base. This is useful for contact forms, support inquiries, FAQ forms, or any scenario where you want an immediate, contextual reply.

### How Smart Replies Work

1. A submission arrives and passes validation and spam checks.
2. Postbox reads the submission data and the linked knowledge base.
3. An AI-generated reply is created based on the submission content and knowledge base.
4. What happens next depends on `smart_replies_mode`:

**Draft mode** (`"draft"`, default): The reply is generated and stored on the submission. It's visible in the dashboard and via API/webhooks, but not sent anywhere. You review it and decide what to do with it. This is the safer option.

**Auto mode** (`"auto"`): Postbox looks for an email field in the submission data. If one is found, Postbox sends the generated reply directly to that email address from `{form_name} <hey@usepostbox.com>` with subject `"Re: Your submission to {form_name}"`. If no email field exists in the submission, the reply is stored with status `"completed"` (same as draft mode). The form owner is never bypassed on the notification side; webhooks and connectors still fire regardless.

### Setting Up Smart Replies

1. Create a knowledge base with relevant content (see Knowledge Bases section)
2. Create or update a form with:
   - `smart_reply_enabled: true`
   - `knowledge_base_id: "{uuid-of-your-knowledge-base}"`
   - `smart_replies_mode: "draft"` or `"auto"`
3. Submissions to that form will now generate replies automatically

### Smart Reply Data in Submissions

When you retrieve a submission, smart reply data appears in these fields:

| Field | Description |
|-------|-------------|
| `reply_status` | `"pending"` (not yet generated), `"completed"` (reply generated and ready), `"sent"` (auto-sent to submitter via email), `"failed"` (email delivery failed) |
| `reply_content` | The generated reply text, or `null` if not yet generated |
| `reply_reason` | Human-readable context when reply is not sent. E.g., `"no email field found in submission"` (completed), or error details (failed). `null` when pending or sent. |
| `replied_at` | ISO 8601 timestamp of when the reply was sent. Only set when `reply_status` is `"sent"`. `null` otherwise. |

### Credit Cost

Each smart reply costs **$0.01** in AI credits.

## Webhooks

Forms can be configured with webhook URLs for real-time submission delivery. Webhooks are the primary mechanism for building custom workflows on top of Postbox. When a submission completes processing, Postbox sends the full result (including spam analysis, translations, and smart replies) to your endpoint.

### Configuration

Set `webhook_enabled`, `webhook_url`, and `webhook_secret` when creating or updating a form via the API, or configure through the dashboard.

- `webhook_url` must be HTTPS (HTTP allowed only for localhost/127.0.0.1 during development)
- `webhook_secret` must be 8-128 characters, used for HMAC-SHA256 payload signing

### Payload

When a submission passes processing (validation + spam check), Postbox sends a POST request to your webhook URL:

```json
{
  "id": "evt_submission_created_{submission_id}",
  "type": "submission.created",
  "created_at": "2026-03-10T14:30:00Z",
  "data": {
    "form": {
      "id": "uuid",
      "name": "Contact",
      "slug": "contact",
      "visibility": "public",
      "version": 1
    },
    "submission": {
      "id": "uuid",
      "data": {
        "name": "Alice",
        "email": "alice@example.com",
        "message": "Hi there"
      },
      "spam_flag": false,
      "spam_confidence": null,
      "spam_reason": null,
      "processing_status": "completed",
      "processing_reason": null,
      "original_language": null,
      "translated_data": null,
      "reply_status": "pending",
      "reply_content": null,
      "reply_reason": null,
      "replied_at": null,
      "inserted_at": "2026-03-10T14:30:00Z",
      "updated_at": "2026-03-10T14:30:01Z"
    }
  }
}
```

### Signature Verification

Every webhook request includes signing headers for verification:

| Header | Description |
|--------|-------------|
| `webhook-id` | Event ID (e.g. `evt_submission_created_{id}`) |
| `webhook-timestamp` | Unix timestamp (seconds) |
| `webhook-signature` | HMAC-SHA256 signature in format `v1,{base64_digest}` |

To verify, compute `HMAC-SHA256(webhook_secret, "{webhook-id}.{webhook-timestamp}.{body}")` and compare with the signature.

**JavaScript verification example:**

```javascript
const crypto = require("crypto");

function verifyWebhook(req, secret) {
  const webhookId = req.headers["webhook-id"];
  const timestamp = req.headers["webhook-timestamp"];
  const signature = req.headers["webhook-signature"];
  const body = JSON.stringify(req.body);

  const signedContent = `${webhookId}.${timestamp}.${body}`;
  const expectedSignature = crypto
    .createHmac("sha256", secret)
    .update(signedContent)
    .digest("base64");

  const received = signature.replace("v1,", "");
  return crypto.timingSafeEqual(
    Buffer.from(expectedSignature),
    Buffer.from(received),
  );
}
```

**Python verification example:**

```python
import hmac
import hashlib
import base64

def verify_webhook(headers, body, secret):
    webhook_id = headers['webhook-id']
    timestamp = headers['webhook-timestamp']
    signature = headers['webhook-signature']

    signed_content = f"{webhook_id}.{timestamp}.{body}"
    expected = base64.b64encode(
        hmac.new(secret.encode(), signed_content.encode(), hashlib.sha256).digest()
    ).decode()

    received = signature.replace('v1,', '')
    return hmac.compare_digest(expected, received)
```

### Retry and Auto-Disable

- Delivery retries up to 3 attempts on failure
- Success: any HTTP 2xx response
- After 3 consecutive failures, the webhook is automatically disabled and you receive an email alert
- Re-enable through the dashboard or by updating the form via `PUT /api/forms/{id}` with `"webhook_enabled": true`. This resets the failure counter so the webhook starts fresh.

## Connectors (Discord and Slack)

Postbox can deliver submission notifications directly to Discord and Slack channels. Connectors are managed through the dashboard.

### How It Works

1. **Get a webhook URL from Discord or Slack:**
   - **Discord:** Go to your server > Channel Settings > Integrations > Webhooks > New Webhook > Copy Webhook URL
   - **Slack:** Go to https://api.slack.com/apps > Create New App > Incoming Webhooks > Activate > Add New Webhook to Workspace > Copy Webhook URL
2. Create a connector in the Postbox dashboard by pasting the webhook URL
3. Link the connector to one or more forms
4. New submissions trigger formatted notifications to your channels

### Discord Notifications

Discord notifications use rich embeds with:

- Form name as the title
- Submission fields displayed inline
- Spam flag and reason if applicable
- Translated data section (if translation is enabled)
- Color-coded: indigo for normal submissions, red for spam

### Slack Notifications

Slack notifications use Block Kit formatting with:

- Header block with form name
- Warning section for spam-flagged submissions
- Submission fields in a structured layout
- Translated data section (if translation is enabled)
- Footer with form slug

### Delivery

- Retries up to 3 attempts on failure
- Auto-disables after 3 consecutive failures (with email alert)
- Can be paused and re-enabled through the dashboard
- Delivery is independent of custom webhooks (both can run simultaneously)

## MCP Server

Connect Postbox to AI assistants to query forms, analyze submissions, and manage data conversationally. Requires Pro plan.

### Connection

```
Endpoint: https://usepostbox.com/mcp
Transport: StreamableHTTP
Auth: OAuth 2.1 (Authorization Code + PKCE)
```

### Connecting in Claude.ai / ChatGPT

1. Open MCP client settings
2. Add a new remote MCP server with URL: `https://usepostbox.com/mcp`
3. The client redirects you to Postbox to authorize
4. Sign in and approve - connected

### MCP Client Configuration (CLI tools)

For Claude Code, Cursor, or CLI-based MCP clients:

```json
{
  "mcpServers": {
    "postbox": {
      "type": "http",
      "url": "https://usepostbox.com/mcp"
    }
  }
}
```

The client handles the OAuth flow automatically on first connection.

### Available Tools

All tools are scoped to the authenticated user's account.

#### list_forms

List all your forms.

Parameters: none

Returns: Array of form summaries (id, name, slug, visibility, submission_count, created_at).

#### get_form

Get form details including field schema and AI settings.

Parameters:
- `form_id` (string, required): The form UUID

Returns: Form detail with fields_schema, spam settings, localisation, smart reply config.

#### list_submissions

List submissions for a form with optional filtering.

Parameters:
- `form_id` (string, required): The form UUID
- `filter` (string, optional): `"inbox"` (default), `"spam"`, or `"all"`
- `limit` (integer, optional): Max results, default 50

Returns: Array of submissions (id, data, spam_flag, processing_status, created_at).

#### get_submission

Get full details of a single submission.

Parameters:
- `submission_id` (string, required): The submission UUID

Returns: Full submission detail including data, spam analysis, translations, and processing status.

#### get_dashboard_stats

Get overview statistics for your account.

Parameters: none

Returns: `{ total_forms, total_submissions, spam_blocked, ai_replies }`

#### translate_submission

Translate a submission on demand. Returns existing translation if already translated. Uses AI credits.

Parameters:
- `submission_id` (string, required): The submission UUID

Returns: Translated data with detected source language.

#### analyze_spam

Run AI spam analysis on a submission. Returns existing analysis if already checked. Uses AI credits.

Parameters:
- `submission_id` (string, required): The submission UUID

Returns: Spam analysis (flagged, confidence, reason).

#### draft_reply

Generate a reply for a submission using a knowledge base. Returns existing reply if already generated. Uses AI credits.

Parameters:
- `submission_id` (string, required): The submission UUID
- `knowledge_base_id` (string, optional): Falls back to form's linked knowledge base.

Returns: Generated reply content.

#### summarize_submissions

Get a summary of recent submissions for a form with patterns and insights.

Parameters:
- `form_id` (string, required): The form UUID
- `limit` (integer, optional): Number of recent submissions to analyze. Default: 50, max: 200.

Returns: Summary including total submissions, spam count, replied count, date range, and recent submission data.

## Error Reference

### Form API Errors

Form management endpoints (`/api/forms/*`) use this format:

| Status | Scenario | Body Format |
|--------|----------|-------------|
| 401 | Invalid or missing API key | `Unauthorized` (plain text) |
| 404 | Form not found | `{ "errors": "Form not found" }` |
| 422 | Form validation errors | `{ "errors": { "field": ["message"] } }` |

### Submission API Errors

Authenticated submission endpoints (`/api/forms/{id}/submissions/*`):

| Status | Scenario | Body Format |
|--------|----------|-------------|
| 400 | Invalid pagination parameters | `{ "error": { "code": 400, "message": "Invalid pagination parameters" } }` |
| 404 | Form or submission not found | `{ "error": { "code": 404, "message": "Form not found" } }` |
| 422 | Could not delete submission | `{ "error": { "code": 422, "message": "Could not delete submission" } }` |

### Public Submission Endpoint Errors

Public submission endpoint (`POST /api/{opaque_segment}/f/{slug}`):

| Status | Scenario | Body Format |
|--------|----------|-------------|
| 400 | Validation error | `{ "error": { "code": "validation_error", "message": "Validation failed", "details": { "field": ["message"] } } }` |
| 401 | Missing submission auth for private form | `{ "error": { "code": "unauthorized", "message": "..." } }` |
| 404 | Form not found | `{ "error": { "code": "form_not_found", "message": "Not found" } }` |
| 429 | Submission limit reached (free plan) | `{ "error": { "code": "plan_limit_exhausted", "message": "...", "upgrade_url": "https://usepostbox.com/settings/billing" } }` |

## Rate Limits

| Endpoint | Limit |
|----------|-------|
| Submission (`POST /api/{opaque_segment}/f/{slug}`) | 10 per minute per IP |
| API (`/api/*`) | 60 per minute per API key |
| MCP (`/mcp`) | 30 per minute per API key |

Rate-limited responses return `429 Too Many Requests` with `retry_after` in seconds.

Proactive rate limit headers (e.g., `X-RateLimit-Remaining`) are not included on successful responses. Implement backoff based on `429` responses and the `retry_after` value.

## Pricing

- **Free**: 1 form, 5,000 lifetime submissions, 50 AI credits (one-time)
- **Pro** ($19/mo or $199/yr): Unlimited forms and submissions, MCP access, 500 AI credits/month (replenishing), metered overflow after credits are exhausted

**Metered billing (Pro only):** When a Pro user's 500 monthly AI credits are exhausted, AI features continue working seamlessly. Usage is metered at the per-credit rates listed below and billed at the end of the billing cycle. There is no interruption to service. The `X-Postbox-Metered: "true"` response header indicates when metered billing is active.

**Free plan:** When the 50 one-time credits are exhausted, AI features (intelligent spam, translation, smart replies) stop working. Standard spam protection (heuristics, honeypot, content moderation) is always free and unaffected.

AI credit costs: spam detection $0.005/submission, translation $0.005/submission, smart reply $0.01/reply.

Standard spam protection (heuristics, honeypot, content moderation) is always free and does not use credits.

## Integration Guide

### Always Use fetch, Never `<form action=...>`

When integrating a Postbox form into a frontend, always submit via JavaScript `fetch()`. Never use a plain HTML `<form method="POST" action="...">`.

**Why this matters:** A `<form action>` submission triggers a full page navigation. The browser leaves your page, and you have zero ability to handle what happens next. If Postbox returns a 400 validation error with per-field details (like `{"email": ["is required"]}`), that structured error response is lost. The user sees a raw JSON page or a browser error. You can't show inline validation messages, you can't keep the form state, you can't show a success message in context.

With `fetch()`, you get the full response object. You can parse the `error.details` object, map errors to specific fields, show them inline, and let the user correct and resubmit without losing their input. This is the only acceptable integration pattern.

### HTML + JavaScript

```html
<form id="contact-form">
  <input type="text" name="name" required placeholder="Name" />
  <input type="email" name="email" required placeholder="Email" />
  <textarea name="message" required placeholder="Message"></textarea>
  <!-- honeypot field: hidden from users, traps bots -->
  <div style="position:absolute; left:-9999px; top:-9999px; opacity:0; pointer-events:none;">
    <input type="text" name="website" tabindex="-1" autocomplete="off" />
  </div>
  <button type="submit">Send</button>
  <div id="form-errors" style="display:none; color:red;"></div>
  <div id="form-success" style="display:none; color:green;">
    Submitted successfully!
  </div>
</form>

<script>
  document
    .getElementById("contact-form")
    .addEventListener("submit", async (e) => {
      e.preventDefault();
      const errorsEl = document.getElementById("form-errors");
      const successEl = document.getElementById("form-success");
      errorsEl.style.display = "none";
      successEl.style.display = "none";

      const data = Object.fromEntries(new FormData(e.target));

      try {
        const res = await fetch("https://usepostbox.com/api/{opaque_segment}/f/contact", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(data),
        });

        if (res.ok) {
          successEl.style.display = "block";
          e.target.reset();
        } else {
          const body = await res.json();
          if (body.error?.details) {
            const messages = Object.entries(body.error.details)
              .map(([field, errs]) => `${field}: ${errs.join(", ")}`)
              .join("\n");
            errorsEl.textContent = messages;
          } else {
            errorsEl.textContent =
              body.error?.message || "Something went wrong.";
          }
          errorsEl.style.display = "block";
        }
      } catch (err) {
        errorsEl.textContent = "Network error. Please try again.";
        errorsEl.style.display = "block";
      }
    });
</script>
```

### cURL

```bash
curl -X POST https://usepostbox.com/api/{opaque_segment}/f/contact \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "message": "Hello"}'
```

### Python - Create Form + Submit

```python
import os, requests

api_key = os.environ["POSTBOX_API_TOKEN"]
headers = {"Authorization": f"Bearer {api_key}", "Content-Type": "application/json"}

# Create form
form = requests.post("https://usepostbox.com/api/forms", json={
    "name": "Feedback",
    "slug": "feedback",
    "visibility": "public",
    "fields_schema": {
        "fields": [
            {"name": "email", "type": "email", "required": True},
            {"name": "rating", "type": "number", "required": True},
            {"name": "comment", "type": "string", "required": False}
        ]
    }
}, headers=headers).json()

endpoint = form["endpoint"]

# Submit data (no auth needed for public forms)
response = requests.post(endpoint, json={
    "email": "user@example.com",
    "rating": 5,
    "comment": "Works great"
})

if response.ok:
    print("Submitted:", response.json())
else:
    print("Error:", response.json())
```

### JavaScript/Node.js - Create Form + Submit

```javascript
const apiKey = process.env.POSTBOX_API_TOKEN;

// Create form
const form = await fetch("https://usepostbox.com/api/forms", {
  method: "POST",
  headers: {
    Authorization: `Bearer ${apiKey}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    name: "Feedback",
    slug: "feedback",
    visibility: "public",
    fields_schema: {
      fields: [
        { name: "email", type: "email", required: true },
        { name: "rating", type: "number", required: true },
        { name: "comment", type: "string", required: false },
      ],
    },
  }),
}).then((r) => r.json());

// Submit data (no auth needed)
const res = await fetch(form.endpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    email: "user@example.com",
    rating: 5,
    comment: "Works great",
  }),
});

if (res.ok) {
  console.log("Submitted:", await res.json());
} else {
  console.log("Error:", await res.json());
}
```

### Agent Discovery Pattern

Postbox is designed for agent-native data collection. AI agents don't need an SDK or documentation. They can discover what a form expects and submit data in two HTTP calls.

**Step 1: Discover the schema**

The agent sends a GET request to the form's endpoint to learn what fields are expected:

```bash
GET https://usepostbox.com/api/{opaque_segment}/f/contact
```

Response:

```json
{
  "name": "Contact",
  "slug": "contact",
  "fields": [
    { "name": "name", "type": "string", "required": true },
    { "name": "email", "type": "email", "required": true },
    { "name": "message", "type": "string", "required": false }
  ]
}
```

The agent now knows: there are 3 fields, `name` and `email` are required, `email` must be a valid email, and `message` is optional.

**Step 2: Validate locally and submit**

The agent constructs a payload matching the schema and submits:

```bash
POST https://usepostbox.com/api/{opaque_segment}/f/contact
Content-Type: application/json

{"name": "Agent", "email": "agent@example.com", "message": "Automated submission"}
```

**Why this matters:** This discover-then-submit pattern means any AI agent, script, or automation can integrate with any Postbox form without prior configuration, hardcoded field names, or an SDK. The form owner defines the schema once; consumers discover it at runtime.

**For agent builders:** If you're building an agent that submits to Postbox forms owned by others, always discover the schema first. Don't hardcode field names. Schemas can be versioned and updated by the form owner at any time, and the discovery endpoint always returns the current version.

## Links

- Site: https://usepostbox.com
- Documentation: https://docs.usepostbox.com
- Features: https://usepostbox.com/features
- Pricing: https://usepostbox.com/pricing
- Blog: https://usepostbox.com/blog
- MCP endpoint: https://usepostbox.com/mcp
- API keys: https://usepostbox.com/integrations/api-keys
