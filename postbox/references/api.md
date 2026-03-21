# Postbox - Full API Reference

> Postbox is an agent-native structured data collection platform. Forms are the base unit - a well-defined collection of fields that expresses intent. A contact form can be name, email, message. A bug report form can be title, reproduction steps. Create a form by defining the schema, get an endpoint, and start receiving validated submissions from HTML forms, scripts, or AI agents. No backend to build, no database to manage, no validation to write.

Base URL: `https://usepostbox.com`

Two interfaces:

- **REST API** (Bearer token auth, all plans) - Create forms, collect submissions, configure processing pipelines, and integrate with your existing systems via destinations (webhooks, Discord, Slack).
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
      { "name": "name", "type": "string", "rules": [{ "op": "required" }] },
      { "name": "email", "type": "email", "rules": [{ "op": "required" }] },
      { "name": "message", "type": "string" }
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
| `smart_reply_email_field` | string | no | Name of the field containing the submitter's email address for auto-mode smart replies (e.g., `"email"`). Must match a field name in `fields_schema`. If the form has exactly one email-type field, this is auto-set. Required for auto-mode when the form has multiple email fields. |
| `knowledge_base_id` | string | no | UUID of the knowledge base for smart replies. Required when `smart_reply_enabled` is `true`. See Knowledge Bases section. |

Response `201 Created`:

```json
{
  "form": {
    "id": "uuid",
    "name": "string",
    "slug": "string",
    "visibility": "public|private",
    "submission_token": "string|null",
    "created_at": "ISO 8601",
    "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/{slug}"
  }
}
```

**Important: `submission_token` is only returned for private forms and is shown exactly once in the create response. Store it immediately in a secure location (e.g., environment variable, secrets manager). You will not be able to retrieve it again via the API.** This token is what submitters (or your backend/agent) must include as a Bearer token when posting to a private form. If you lose it, you can generate a new one from the form settings page in the dashboard (this invalidates the old token).

The `endpoint` field is the pre-computed submission URL, ready to use.

Response `422 Unprocessable Entity`:

```json
{ "error": { "code": "validation_error", "message": "Validation failed", "details": { "slug": ["has already been taken"] } } }
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
      "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/{slug}",
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
  "form": {
    "id": "uuid",
    "name": "string",
    "slug": "string",
    "visibility": "public|private",
    "fields_schema": {
      "fields": [{ "name": "email", "type": "email", "rules": [{ "op": "required" }] }]
    },
    "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/{slug}",
    "submission_count": 0,
    "created_at": "ISO 8601"
  }
}
```

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Form not found" } }
```

### Update Form

```
PUT /api/forms/{id}
```

Same fields as Create, all optional. Destinations (webhooks, Discord, Slack) are managed separately via the Destinations API.

**Schema changes produce a new endpoint URL.** When you update `fields_schema`, the response contains a new `endpoint` value. You must update your submission code (frontend form, script, agent) to POST to this new URL. The old URL still works but validates against the old schema. See Schema Versioning for details.

Response `200`: Same shape as Get Form (`{"form": {...}}`, includes the current `endpoint` URL).

### Delete Form

```
DELETE /api/forms/{id}
```

Response `200`: Returns the deleted form (`{"form": {...}}`).

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Form not found" } }
```

## Field Schema

The `fields_schema` object defines what data a form accepts. Each field has a name, type, and an optional `rules` array for validation.

```json
{
  "fields": [
    {
      "name": "field_name",
      "type": "string|email|number|boolean|date",
      "rules": [
        { "op": "required" }
      ]
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

### Field Rules

Rules are an array of validation operators applied to each field. Each rule is an object with an `op` (operator) and optional parameters.

| Operator | Parameters | Applies To | Description |
|----------|-----------|------------|-------------|
| `required` | none | all types | Field must be present and non-empty |
| `honeypot` | none | string | Marks field as a spam trap (see Honeypot Fields) |
| `one_of` | `"values": [...]` | string, number | Value must be one of the listed options |
| `not_one_of` | `"values": [...]` | string, number | Value must not be any of the listed options |
| `min_length` | `"value": n` | string | Minimum string length |
| `max_length` | `"value": n` | string | Maximum string length |
| `min` | `"value": n` | number | Minimum numeric value (inclusive) |
| `max` | `"value": n` | number | Maximum numeric value (inclusive) |
| `pattern` | `"value": "regex"` | string | Must match the regular expression |
| `after` | `"value": "YYYY-MM-DD"` | date | Date must be after the boundary |
| `before` | `"value": "YYYY-MM-DD"` | date | Date must be before the boundary |

**Example with multiple rules:**

```json
{
  "fields": [
    { "name": "email", "type": "email", "rules": [{ "op": "required" }] },
    { "name": "age", "type": "number", "rules": [{ "op": "min", "value": 18 }, { "op": "max", "value": 120 }] },
    { "name": "priority", "type": "string", "rules": [{ "op": "required" }, { "op": "one_of", "values": ["low", "medium", "high"] }] },
    { "name": "message", "type": "string", "rules": [{ "op": "max_length", "value": 500 }] }
  ]
}
```

### Conditional Rules

Any rule can include a `when` clause to make it conditional. The rule only applies if the condition evaluates to true against the submission data.

```json
{
  "name": "company",
  "type": "string",
  "rules": [
    { "op": "required", "when": { "field": "role", "is": "eq", "value": "business" } }
  ]
}
```

Condition comparators:

| Comparator | Description |
|------------|-------------|
| `eq` | Equals (with type coercion for numbers) |
| `neq` | Not equals |
| `one_of` | Value is one of a list (`"value": [...]`) |
| `not_one_of` | Value is not in a list |
| `gt`, `lt`, `gte`, `lte` | Numeric comparisons |
| `filled` | Field is present and non-empty |
| `empty` | Field is absent or empty |

### Honeypot Fields

Honeypot fields are invisible spam traps. Add `{ "op": "honeypot" }` to a field's rules to use it as a honeypot. A field should not have both `required` and `honeypot` rules.

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
{ "name": "website", "type": "string", "rules": [{ "op": "honeypot" }] }
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
      { "name": "employee_id", "type": "string", "rules": [{ "op": "required" }] },
      { "name": "feedback", "type": "string", "rules": [{ "op": "required" }] }
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

If you lose the `submission_token`, go to the form settings page in the Postbox dashboard to generate a new one. This invalidates the previous token immediately.

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

**Idempotency:** Include an `Idempotency-Key` header (any string up to 256 characters, typically a UUID) to make submissions safe to retry. If a submission with the same key already exists for this form, the original submission is returned (200) instead of creating a duplicate. Without the header, every POST creates a new submission. The key is scoped to the form - the same key on different forms creates separate submissions.

```
Idempotency-Key: 550e8400-e29b-41d4-a716-446655440000
```

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

Response `422` (validation error):

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

Returns the form's field schema so agents and scripts can discover what to submit. Honeypot fields are automatically hidden from this response.

Response `200`:

```json
{
  "name": "Contact",
  "slug": "contact",
  "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/contact",
  "method": "POST",
  "content_type": "application/json",
  "visibility": "public",
  "fields": [
    { "name": "name", "type": "string", "rules": [{ "op": "required" }] },
    { "name": "email", "type": "email", "rules": [{ "op": "required" }] },
    { "name": "message", "type": "string" }
  ]
}
```

For private forms, the response includes an `authentication` object:

```json
{
  "authentication": {
    "type": "bearer",
    "header": "Authorization: Bearer <submission_token>",
    "note": "This is a private form. A valid submission token is required to POST data."
  }
}
```

This endpoint is key for agent-native data collection. See Agent Discovery Pattern for details.

### Content Negotiation

The submission endpoint is self-documenting. The same URL serves two purposes based on the request:

- **POST** submits data (as described above)
- **GET with `Accept: application/json`** returns the schema as JSON (see Discover Form Schema above)
- **GET with `Accept: text/html`** (i.e., a browser) renders a documentation page showing the form's fields, types, and endpoint details

One URL, two audiences. Agents get machine-readable schema. Developers opening the URL in a browser get a human-readable reference. This is not a fillable form. It is documentation of what the endpoint accepts.

### List Submissions

```
GET /api/forms/{form_id}/submissions
```

Requires Bearer token authentication. Returns paginated submissions for a form you own, sorted by `created_at` descending (newest first).

Query parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `filter` | string | `"inbox"` | `"inbox"` (non-spam only), `"spam"` (spam only), or `"all"` |
| `search` | string | none | Full-text search across submission data fields |
| `reply_status` | string | none | Filter by reply status: `"pending"`, `"completed"`, `"sent"`, `"failed"` |
| `processing_status` | string | none | Filter by processing status: `"pending"`, `"processing"`, `"completed"`, `"skipped"`, `"failed"` |
| `sort_by` | string | `"inserted_at"` | Sort field: `"inserted_at"` or `"id"` |
| `sort_order` | string | `"desc"` | Sort direction: `"asc"` or `"desc"` |
| `page` | integer | 1 | Page number |
| `page_size` | integer | 20 | Results per page (max 50) |

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
      "reply_subject": null,
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
{ "error": { "code": "not_found", "message": "Form not found" } }
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
  "reply_subject": null,
  "reply_content": null,
  "reply_reason": null,
  "replied_at": null,
  "created_at": "ISO 8601"
}
```

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Submission not found" } }
```

### Delete Submission

```
DELETE /api/forms/{form_id}/submissions/{id}
```

Requires Bearer token authentication.

Response `200`: Returns the deleted submission (same shape as Get Submission).

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Submission not found" } }
```

### Processing Pipeline

The submission endpoint responds `201 Created` immediately. All processing happens async:

1. **Validation** - Schema-enforced. Required fields, types, constraints.
2. **Spam detection** - If enabled, runs your chosen strategy. Spam is flagged silently (submitter always gets `201`).
3. **Translation** - If enabled, detects language and translates. Original preserved.
4. **Smart replies** - If enabled with a knowledge base, generates and optionally sends a reply.
5. **Notifications** - Destinations (webhooks, Discord, Slack) and email notifications are delivered after processing.

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

Destinations fire after processing completes (or fails), so the destination payload always contains the final `processing_status`.

## Schema Versioning

Every change to `fields_schema` via `PUT /api/forms/{id}` auto-creates a new versioned endpoint. This is how Postbox guarantees that existing integrations never break.

**How it works:**

- When you update a form's `fields_schema`, the form's internal version increments automatically.
- The update response (and `GET /api/forms/{id}`) returns the new `endpoint` URL. **Always read the `endpoint` field from the response and update your integration to use it.** The opaque segment in the URL encodes the version, so the URL changes on each schema update.
- Old endpoint URLs keep working. Submissions to an old URL are validated against the schema version that URL was created for. This means existing frontend forms, scripts, and agents continue working without changes.
- Old submissions retain their original shape. If version 1 had fields `[name, email]` and version 2 adds `[name, email, phone]`, version 1 submissions still only contain `name` and `email`.
- You don't need to manage versions manually. Just update the schema whenever you need to, and Postbox handles backward compatibility.

**When to use this:** Change your schema freely as your product evolves. Add new fields, remove old ones, change types. Existing integrations using old endpoint URLs continue working. New integrations should use the latest endpoint URL from the form response to get the current schema validation.

**Note:** There is no API to list or retrieve previous schema versions. Old versions are kept internally for backward compatibility but are not exposed. The API always returns the current version.

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
  "knowledge_base": {
    "id": "uuid",
    "name": "string",
    "content": "string",
    "created_at": "ISO 8601"
  }
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
  "knowledge_base": {
    "id": "uuid",
    "name": "string",
    "content": "string",
    "created_at": "ISO 8601"
  }
}
```

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Knowledge base not found" } }
```

### Update Knowledge Base

```
PUT /api/knowledge_bases/{id}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | no | Updated name |
| `content` | string | no | Updated content |

Response `200`: Same shape as Get Knowledge Base (`{"knowledge_base": {...}}`).

### Delete Knowledge Base

```
DELETE /api/knowledge_bases/{id}
```

Response `200`: Returns the deleted knowledge base (`{"knowledge_base": {...}}`).

**Important:** If a knowledge base is linked to a form with smart replies enabled, deleting it will cause smart reply processing to fail for that form. Update the form to disable smart replies or link a different knowledge base first.

## Smart Replies

Smart replies auto-generate responses to form submissions using a knowledge base. This is useful for contact forms, support inquiries, FAQ forms, or any scenario where you want an immediate, contextual reply.

### How Smart Replies Work

1. A submission arrives and passes validation and spam checks.
2. Postbox reads the submission data and the linked knowledge base.
3. An AI-generated reply is created based on the submission content and knowledge base.
4. What happens next depends on `smart_replies_mode`:

**Draft mode** (`"draft"`, default): The reply is generated and stored on the submission. It's visible in the dashboard and via API/webhooks, but not sent anywhere. You review it and decide what to do with it. This is the safer option.

**Auto mode** (`"auto"`): Postbox uses the `smart_reply_email_field` setting to find the submitter's email in the submission data. If a valid email is found, Postbox sends the generated reply directly to that address. If no email field is configured or the field is empty, the reply is stored with status `"completed"` (same as draft mode). The form owner is never bypassed on the notification side; destinations still fire regardless.

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
| `reply_subject` | The generated email subject line, or `null` if not yet generated |
| `reply_content` | The generated reply text, or `null` if not yet generated |
| `reply_reason` | Human-readable context when reply is not sent. E.g., `"no email field found in submission"` (completed), or error details (failed). `null` when pending or sent. |
| `replied_at` | ISO 8601 timestamp of when the reply was sent. Only set when `reply_status` is `"sent"`. `null` otherwise. |

### Credit Cost

Each smart reply costs **$0.01** in AI credits.

## Destinations

Every form can forward submissions to one or more destinations. Destinations are the primary mechanism for building custom workflows on top of Postbox. When a submission completes processing, Postbox delivers the full result (including spam analysis, translations, and smart replies) to each enabled destination.

Three destination types:

- `webhook` - Raw JSON POST to your endpoint with HMAC-SHA256 signature verification
- `discord` - Formatted as a Discord embed (rich message with fields, colors, and metadata)
- `slack` - Formatted as Slack Block Kit (structured blocks with header, fields, and footer)

### API Endpoints

```
GET    /api/forms/{form_id}/destinations
POST   /api/forms/{form_id}/destinations
DELETE /api/forms/{form_id}/destinations/{id}
POST   /api/forms/{form_id}/destinations/{id}/regenerate-secret
```

All destination endpoints require Bearer token authentication.

### List Destinations

```
GET /api/forms/{form_id}/destinations
```

Response `200`:

```json
{
  "destinations": [
    {
      "id": "uuid",
      "type": "webhook",
      "name": "My Backend",
      "url": "https://example.com/webhooks/postbox",
      "enabled": true,
      "last_delivered_at": "ISO 8601|null",
      "failure_count": 0,
      "last_error": "string|null",
      "disabled_at": "ISO 8601|null"
    },
    {
      "id": "uuid",
      "type": "discord",
      "name": "Team Alerts",
      "url": "https://discord.com/api/webhooks/...",
      "enabled": true,
      "last_delivered_at": null,
      "failure_count": 0,
      "last_error": null,
      "disabled_at": null
    }
  ]
}
```

### Create Destination

```
POST /api/forms/{form_id}/destinations
```

Request body:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | string | yes | `"webhook"`, `"discord"`, or `"slack"` |
| `name` | string | yes | Human-readable name for this destination |
| `url` | string | yes | The destination URL. For webhook type, must be HTTPS (HTTP allowed only for localhost/127.0.0.1 during development). For Discord/Slack, use the platform webhook URL. |

Request example:

```json
{
  "type": "webhook",
  "name": "My Backend",
  "url": "https://example.com/webhooks/postbox"
}
```

Response `201 Created`:

```json
{
  "destination": {
    "id": "uuid",
    "type": "webhook",
    "name": "My Backend",
    "url": "https://example.com/webhooks/postbox",
    "enabled": true,
    "secret": "whsec_...",
    "last_delivered_at": null,
    "failure_count": 0,
    "last_error": null,
    "disabled_at": null
  }
}
```

The `secret` field is only returned for `webhook` type destinations and is shown only once at creation time. Store it immediately. Discord and Slack destinations do not have a secret.

### Delete Destination

```
DELETE /api/forms/{form_id}/destinations/{id}
```

Response `200`: Returns the deleted destination (wrapped in `{"destination": {...}}`).

Response `404`:

```json
{ "error": { "code": "not_found", "message": "Destination not found" } }
```

### Regenerate Secret

```
POST /api/forms/{form_id}/destinations/{id}/regenerate-secret
```

Generates a new HMAC signing secret for a webhook destination. The old secret is invalidated immediately. Only applies to `webhook` type destinations.

Response `200`:

```json
{
  "destination": {
    "id": "uuid",
    "type": "webhook",
    "name": "My Backend",
    "url": "https://example.com/webhooks/postbox",
    "enabled": true,
    "secret": "whsec_...",
    "last_delivered_at": "ISO 8601|null",
    "failure_count": 0,
    "last_error": null,
    "disabled_at": null
  }
}
```

### Webhook Payload

When a submission passes processing (validation + spam check), Postbox sends a POST request to each enabled webhook destination:

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

### Webhook Signature Verification

Every webhook destination request includes signing headers for verification:

| Header | Description |
|--------|-------------|
| `webhook-id` | Event ID (e.g. `evt_submission_created_{id}`) |
| `webhook-timestamp` | Unix timestamp (seconds) |
| `webhook-signature` | HMAC-SHA256 signature in format `v1,{base64_digest}` |

To verify, compute `HMAC-SHA256(secret, "{webhook-id}.{webhook-timestamp}.{body}")` and compare with the signature. Reject requests where `webhook-timestamp` is older than 5 minutes to prevent replay attacks.

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

### Discord Notifications

Discord destinations format submissions as rich embeds with:

- Form name as the title
- Submission fields displayed inline
- Spam flag and reason if applicable
- Translated data section (if translation is enabled)
- Color-coded: indigo for normal submissions, red for spam

No secret or signature verification needed for Discord destinations.

### Slack Notifications

Slack destinations format submissions as Block Kit messages with:

- Header block with form name
- Warning section for spam-flagged submissions
- Submission fields in a structured layout
- Translated data section (if translation is enabled)
- Footer with form slug

No secret or signature verification needed for Slack destinations.

### Retry and Auto-Disable

- Delivery retries up to 3 attempts on failure (applies to all destination types)
- Success: any HTTP 2xx response
- After 3 consecutive failures, the destination is automatically disabled and you receive an email alert
- Re-enable through the dashboard. This resets the failure counter so the destination starts fresh.

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
List all your forms. Parameters: none. Returns: Array of form summaries.

#### get_form
Get form details including field schema and AI settings. Parameters: `form_id` (string, required).

#### list_submissions
List submissions for a form with optional filtering. Parameters: `form_id` (string, required), `filter` (optional), `limit` (optional, default 50).

#### get_submission
Get full details of a single submission. Parameters: `submission_id` (string, required).

#### get_dashboard_stats
Get overview statistics for your account. Parameters: none. Returns: `{ total_forms, total_submissions, spam_blocked, ai_replies }`.

#### translate_submission
Translate a submission on demand. Returns existing translation if already translated. Uses AI credits. Parameters: `submission_id` (string, required).

#### analyze_spam
Run AI spam analysis on a submission. Returns existing analysis if already checked. Uses AI credits. Parameters: `submission_id` (string, required).

#### draft_reply
Generate a reply for a submission using a knowledge base. Returns existing reply if already generated. Uses AI credits. Parameters: `submission_id` (string, required), `knowledge_base_id` (string, optional).

#### summarize_submissions
Get a summary of recent submissions for a form with patterns and insights. Parameters: `form_id` (string, required), `limit` (optional, default 50, max 200).

## Error Reference

All API errors use a consistent format:

```json
{ "error": { "code": "string_code", "message": "Human-readable message" } }
```

Validation errors include a `details` field with per-field messages. Some errors include extra fields like `upgrade_url` or `retry_after`.

### Authenticated API Errors

| Status | Code | Scenario |
|--------|------|----------|
| 401 | `unauthorized` | Invalid or missing API key |
| 403 | `form_limit_reached` | Free plan form limit (includes `upgrade_url`) |
| 403 | `pro_required` | Feature requires Pro plan (includes `upgrade_url`) |
| 404 | `not_found` | Resource not found |
| 422 | `validation_error` | Validation failed (includes `details` with per-field errors) |
| 422 | `delete_failed` | Could not delete resource |
| 422 | `invalid_destination_type` | Operation not supported for this destination type |
| 400 | `invalid_params` | Invalid pagination or filter parameters |
| 429 | `rate_limited` | Rate limit exceeded (includes `retry_after`) |

### Public Submission Endpoint Errors

| Status | Code | Scenario |
|--------|------|----------|
| 422 | `validation_error` | Schema validation failed (includes `details`) |
| 401 | `unauthorized` | Missing or invalid submission token for private form |
| 401 | `invalid_token` | Malformed endpoint token |
| 404 | `form_not_found` | Form not found |
| 429 | `plan_limit_exhausted` | Submission limit reached on free plan (includes `upgrade_url`) |

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

**Why this matters:** A `<form action>` submission triggers a full page navigation. The browser leaves your page, and you have zero ability to handle what happens next. If Postbox returns a validation error with per-field details (like `{"email": ["is required"]}`), that structured error response is lost. The user sees a raw JSON page or a browser error. You can't show inline validation messages, you can't keep the form state, you can't show a success message in context.

With `fetch()`, you get the full response object. You can parse the `error.details` object, map errors to specific fields, show them inline, and let the user correct and resubmit without losing their input. This is the only acceptable integration pattern.

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
  "endpoint": "https://usepostbox.com/api/{opaque_segment}/f/contact",
  "method": "POST",
  "content_type": "application/json",
  "visibility": "public",
  "fields": [
    { "name": "name", "type": "string", "rules": [{ "op": "required" }] },
    { "name": "email", "type": "email", "rules": [{ "op": "required" }] },
    { "name": "message", "type": "string" }
  ]
}
```

The agent now knows: there are 3 fields, `name` and `email` are required (via the `required` rule), `email` must be a valid email, and `message` is optional (no rules). The response also includes the ready-to-use `endpoint` URL.

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
