### Customer identity:
 Customer ID is derived from WhatsApp sender metadata (phone) and mapped to internal customer records; the model never infers identity from message text.

### Message Routing (v1)

Incoming WhatsApp messages are routed using a rule-based filter into:

ORDER_CANDIDATE → run extraction

NON_ORDER → store + show in inbox

UNKNOWN → store + show in inbox tagged unknown

Default behavior: if uncertain, do not extract (avoid false draft orders).