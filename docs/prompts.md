# Prompt v1.2
Instructions

Respond ONLY with valid JSON matching the schema below.
Output JSON only (no extra text, no explanations).

Do NOT invent information.
Use only what is explicitly present in the message.

Return REAL values (numbers/strings), not type labels like "number" or "string".

REQUIRED:

items (must contain at least 1 item)

quantity for each item

Delivery logic (v1):

If the order appears to be a delivery order and no delivery location is provided, add "delivery_area" to missing_fields.

If delivery vs pickup is unclear, treat as delivery and require delivery location.

If any required or delivery-related information is missing or ambiguous:

add the field name to missing_fields

set status to "NEEDS_REVIEW"

set confidence to less than 0.8

If missing_fields is empty:

set status to "DRAFT"

Use null for optional fields not provided.

Put urgency phrases (e.g., "عاجل", "urgent") into the order-level notes.

Customer identity rule:

If customer_id is not explicitly present in the message, set "customer_id": null.

Do NOT infer or guess customer identity.

Schema (Intermediate Extraction Format)
{
  "customer_id": null,
  "status": "DRAFT | NEEDS_REVIEW",
  "items": [
    {
      "name": "...",
      "quantity": 0,
      "notes": null
    }
  ],
  "delivery_area": null,
  "courier_name": null,
  "notes": null,
  "confidence": 0.0,
  "missing_fields": []
}

Input Message
"Whatsapp message"