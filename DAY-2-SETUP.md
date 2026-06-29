# Day 2 — n8n Fluency + the Company Brain

Yesterday the SOP was hardcoded inside the API node — you noticed how awkward that was to edit. Today you fix it: the company brain moves *into* the workflow as data, and you learn the expressions that wire it in. You'll also meet triggers and error handling. ~4–5 hours.

**What changes vs Day 1**
- Skills (SOP, brand voice, output format) now live in one **Skill Library** node — edit logic in one place, never touch the API node again.
- The request body is built with **expressions** instead of a hardcoded string.
- The API node now **retries on failure** and routes failures to a graceful **error branch** instead of crashing.

---

## Part 1 — Import the v2 workflow (~10 min)
1. n8n → **Workflows → Import from File** → `workflows/lead-qualifier-v2.json`.
2. You'll see: `Start → Skill Library → Sample Lead → Claude → (Extract Answer / Handle Error)`.
3. Re-select your **Anthropic Header Auth credential** on the *Claude - Qualify Lead* node (same one from Day 1).

---

## Part 2 — Meet the Skill Library (~15 min)
Open the **Skill Library** node. The three skill files from Day 1 now live here as three fields: `sop`, `brand_voice`, `output_schema`.

This is the upgrade: **your agent's logic is now data, not code.** To change how it qualifies leads, you edit text here — not the API request.

---

## Part 3 — The expressions lesson (~30 min, the core skill today)
Open **Claude - Qualify Lead** → look at the **JSON** body. Unlike Day 1's hardcoded text, the whole body is now one expression that *builds* the request. Two things to understand:

**1. `$json` vs `$('Node Name')`**
- `$json.lead_name` → a field from the **node immediately upstream** (Sample Lead).
- `$('Skill Library').item.json.sop` → reaches **back to any earlier node by name**, even if that data isn't in the current item anymore.

This matters: data gets replaced as it flows downstream (the Sample Lead node dropped the skill fields). Reaching back by node name is how you grab anything from earlier in the chain. **This single concept unlocks 80% of n8n.**

**2. Why the body is built as an object expression**
The skills contain quotes and line breaks. If you pasted them into raw JSON text (Day 1 style), they'd break the request. By building a JS object (`{{ ({ ... }) }}`), n8n serializes it to valid JSON for you — escaping handled automatically. You'll reuse this pattern constantly.

> Cheat sheet for the syntax: see `n8n-expressions-cheatsheet.md`.

---

## Part 4 — Run it, then prove skills-as-data works (~20 min)
1. **Execute workflow.** Confirm **Extract Answer** shows the JSON output (tier / reasoning / opener).
2. Now open **Skill Library** and make ONE change — e.g. add `- agencies and consultancies` to the disqualifiers in `sop`, or add `do not use the word "platform"` to `brand_voice`.
3. Re-run. The output changes — **and you never touched the API node.** That's the whole point of context engineering: behavior lives in editable skills, not buried in code.

---

## Part 5 — Error handling (~20 min)
The Claude node is now production-minded:
- **Retry on fail:** 3 attempts, 2s apart (handles transient API blips).
- **Error output:** if it still fails, the item routes to **Handle Error** (the second output) instead of killing the run.

**Test it:** temporarily change your API key to something wrong → run → watch the item flow to *Handle Error* with a clean `FAILED` status. Fix the key. This is the "agent fails gracefully, a human gets flagged" pattern hirers look for. Put the right key back when done.

---

## Part 6 — Triggers (guided exercise, ~30 min)
Right now a human clicks "Execute." Real agents react to events. Swap the trigger:
1. Add a **Webhook** node (HTTP Method: POST). Connect it into **Skill Library** (same as the manual trigger — two entry points, one chain).
2. Click **Listen for Test Event**, then send a sample lead to the test URL. Quick ways:
   - Use a tool like Postman/Hoppscotch, or a terminal:
     ```bash
     curl -X POST "<your-test-webhook-url>" \
       -H "Content-Type: application/json" \
       -d '{"lead_name":"Sam Rivera","lead_company":"Northgate SaaS","lead_role":"Head of Growth","lead_note":"Asked about pricing and attribution"}'
     ```
3. **Key gotcha:** a webhook wraps incoming data under `body`. So a real webhook flow references `$json.body.lead_name`, not `$json.lead_name`. (For today, the manual + Sample Lead path keeps testing simple — the webhook is to *understand* how leads arrive in production.)

Triggers worth knowing: **Manual** (testing), **Webhook** (real-time events like form fills), **Schedule** (run every hour/day — great for batch enrichment or reporting).

---

## Ship (~15 min)
1. Download the workflow from n8n (with your edits) and save as `workflows/lead-qualifier-v2.json`.
2. Commit: `Day 2: skills-as-data, expressions, error handling`.
3. ✅ Done.

## What you learned today
- The `$('Node Name')` reach-back pattern — the most important expression in n8n.
- Building request bodies as object expressions (no more broken JSON).
- Context engineering: agent behavior as editable data, not code.
- Production error handling: retries + a graceful failure branch.
- Triggers: manual vs webhook vs schedule, and the `body` wrapping gotcha.

## Watch-outs
- After import, **re-select the Anthropic credential** — credentials never travel inside a workflow file.
- Confirm the model string (`claude-sonnet-4-6`) is current in Anthropic's docs; swap to `claude-haiku-4-5-20251001` to cut test costs.
- If the body expression errors, check you didn't accidentally delete a `+`, a quote, or the closing `}) }}`.

## Tomorrow (Day 3)
APIs, webhooks, and JSON for real — connect a live external data source (enrichment) and start directing Cursor/Claude to write the glue code for you.
