# WhatsApp Order Extraction — v1 Design Document

## Problem

We receive unstructured customer orders via WhatsApp and need to convert them into structured JSON for our order processing system.

**Inputs:**
- Informal conversational text in Arabic, English, or mixed
- Missing information, typos, emojis
- Multiple messages forming a single order
- Reference orders: "Same as last time"

**Outputs:**
- Structured JSON matching our server schema
- Clear flags when information is missing or uncertain

**Known Failure Cases:**
- "Same as last time" / "Not like last time"
- Urgency without details: "Do it fast I am starving"
- Typos: "piza", "buger"
- Emoji-heavy messages
- Incomplete information across multiple messages

---

## Schema (v1)

Our server expects this format:

```json
{
  "customerId": "number",
  "courierName": "string | null",
  "notes": "string | null",
  "products": [
    { "id": "number", "quantity": "number" }
  ]
}
```

For extraction and review purposes, we'll use an **intermediate extraction format** before mapping to the server schema:

```json
{
  "customer_id": "number",
  "status": "DRAFT | NEEDS_REVIEW",
  "items": [
    {
      "name": "string",
      "quantity": "number",
      "notes": "string | null"
    }
  ],
  "courier_name": "string | null",
  "notes": "string | null",
  "confidence": "number (0.0-1.0)",
  "missing_fields": ["string"]
}
```

**Required Fields:**
- `items` (at least one)
- `quantity` (for each item)

**Optional Fields:**
- `courier_name`
- `notes` (item-level or order-level)

---

## Extraction Rules

The LLM receives these instructions:

```
Respond ONLY with valid JSON matching the schema.
- Do not invent information
- If a detail is missing, include it in "missing_fields" array
- Set confidence < 0.8 if uncertain about any field
- Use null for optional fields not provided
```

---

## Handling Missing Data

When required fields are missing:
1. Set `status: "NEEDS_REVIEW"`
2. Populate `missing_fields` array
3. Set `confidence` appropriately (< 0.8)
4. Human agent reviews and contacts customer if needed

**No orders are auto-confirmed in v1.** All orders require human confirmation.

---

## Handling "Same as Last Time"

**v1 approach:**
- Extract as: `"notes": "Customer requested: same as last time"`
- Set `status: "NEEDS_REVIEW"`
- Human agent retrieves previous order manually

This ensures safety. This avoids ambiguity regarding which previous order the customer is referencing.

---

## High Level Processing Flow

```
WhatsApp Message(s)
    ↓
LLM Extraction
    ↓
JSON Output (intermediate format)
    ↓
Validation (schema check, confidence threshold)
    ↓
Database Storage
    ↓
Human Review Queue (if status = NEEDS_REVIEW)
    ↓
Order Confirmation & Processing
```

---

## What v1 Does NOT Do

- Does not auto-confirm orders
- Does not guarantee 100% correctness
- Does not handle voice messages or images
- Does not process orders without human confirmation
- Does not train custom models (uses pre-trained LLMs)

---

## Example Transformations

**Example 1: Complete Order**

*Input:*
```
I want 2 pizzas with pepperoni on top and one burger
courier: Bosta
```

*Output:*
```json
{
  "customer_id": 12345,
  "status": "DRAFT",
  "items": [
    {
      "name": "Pizza",
      "quantity": 2,
      "notes": "pepperoni on top"
    },
    {
      "name": "Burger",
      "quantity": 1,
      "notes": null
    }
  ],
  "courier_name": "Bosta",
  "notes": null,
  "confidence": 0.95,
  "missing_fields": []
}
```

**Example 2: Missing Information**

*Input:*
```
شاورما عاجل 3
```

*Output:*
```json
{
  "customer_id": 12346,
  "status": "NEEDS_REVIEW",
  "items": [
    {
      "name": "Shawarma",
      "quantity": 3,
      "notes": null
    }
  ],
  "courier_name": null,
  "notes": "Customer indicated urgency",
  "confidence": 0.65,
  "missing_fields": []
}
```

---

## Constraints

- Requires internet for LLM API
- Arabic and English only (v1)
- Performance depends on LLM availability
- Product IDs must be mapped manually to server schema before order submission

---

## Future Improvements (v2+)

- **RAG**: Ground extraction in actual menu items and customer history to reduce hallucinations
- **Multi-message consolidation**: Merge fragmented orders within 5-minute windows
- **Confidence-based routing**: Different review workflows based on confidence scores
- **Auto-confirmation**: For high-confidence orders (>0.95)
- **Custom fine-tuning**: Domain-specific model for local dish names and abbreviations

---

**Document Version:** 1.0  
**Last Updated:** January 2026