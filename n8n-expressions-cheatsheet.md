# n8n Expressions Cheat Sheet

Expressions go inside `{{ }}`. A field that starts with `=` is in expression mode. Keep this handy — you'll use these daily.

## Referencing data
| You want | Expression |
|---|---|
| A field from the node right before this one | `{{ $json.field_name }}` |
| A field from a specific earlier node (by name) | `{{ $('Node Name').item.json.field_name }}` |
| All items from an earlier node | `{{ $('Node Name').all() }}` |
| A nested field | `{{ $json.body.lead_name }}` |
| Output of the workflow's trigger | `{{ $json }}` (on the first node) |

> **The one to memorize:** `$('Node Name').item.json.x` reaches back to *any* prior node, even after the data left the current item. This is how you avoid "where did my field go?" problems.

## Building strings
```js
"Hello " + $json.first_name + ", from " + $json.company
```
Use `\n` for line breaks inside a JS string: `"Line 1\nLine 2"`.

## Building a JSON body (the API-call pattern)
Set the body field to expression mode and return an object — n8n serializes it safely:
```js
={{ ({
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "system": $('Skill Library').item.json.sop,
  "messages": [ { "role": "user", "content": $json.user_text } ]
}) }}
```
Why an object instead of raw text: quotes and newlines in your data would break hand-written JSON. Returning an object lets n8n handle all escaping.

## Handy helpers
| Need | Expression |
|---|---|
| Fallback if a field is empty | `{{ $json.name || 'there' }}` |
| Lowercase / trim | `{{ $json.email.toLowerCase().trim() }}` |
| Does text contain something | `{{ $json.note.includes('pricing') }}` |
| Today's date (ISO) | `{{ $now.toISO() }}` |
| Parse a JSON string into an object | `{{ JSON.parse($json.claude_output) }}` |
| Number from text | `{{ Number($json.amount) }}` |
| Conditional value | `{{ $json.tier === 'A' ? 'route to AE' : 'nurture' }}` |

## Gotchas
- **Webhook data is under `body`:** use `$json.body.field`, not `$json.field`.
- **A Set node drops fields it doesn't assign** (unless "Include Other Input Fields" is on). Reach back with `$('Node Name')` instead.
- **`.item` vs `.all()`:** `.item` = the single matching item; `.all()` = the array of all items.
- **Expression mode:** if `{{ }}` shows up as literal text in the output, the field isn't in expression mode — toggle it (or start the value with `=`).
