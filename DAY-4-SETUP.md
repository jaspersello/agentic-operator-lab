# Day 4 — Project 1 Takes Shape: Architecture + a Real Output

Until now your agent produced an answer and... that was it. Today it gets a **destination**: every qualified lead becomes a row in a Google Sheet acting as a simple CRM. You'll also step back and see the whole thing as one architecture. ~5 hours.

**What's new in this version**
- A **Parse Claude JSON** node splits Claude's answer into clean, separate fields (tier, reasoning, opener) — ready to become spreadsheet columns
- A **Google Sheets** node logs each lead as a row (you add this one yourself — it's a good skill, and your sheet + Google login can't travel inside a workflow file anyway)
- The `PROJECT-1.md` file with a proper architecture diagram for your README

---

## Part 1 — Import the workflow (~10 min)
1. n8n → new workflow → **Import from File** → `workflows/project-1-lead-qualifier.json`
2. Re-select your **Anthropic credential** on the Claude node
3. Run it once to confirm it works up to **Parse Claude JSON**

---

## Part 2 — Understand Parse Claude JSON (~20 min)
Open the **Parse Claude JSON** node. This solves a problem: Claude returns its whole answer as *one string* of JSON text. To put `tier` in one column and `reasoning` in another, you first have to turn that string into a real object.

Look at the `tier` field expression:
```
={{ JSON.parse($json.claude_raw.replace(/```json/g, '').replace(/```/g, '').trim())['tier'] }}
```
Breaking it down (this is a great one to paste into Cursor and ask "explain each part"):
- `$json.claude_raw` → Claude's raw answer string
- `.replace(...)` → strips markdown code fences if Claude added any (it sometimes wraps JSON in ```)
- `JSON.parse(...)` → turns the string into a real object
- `['tier']` → grabs just that field

Each output field (tier, reasoning, opener, enrichment_used) does this and pulls out its own piece. Now you have clean columns.

> This is exactly the JSON-parsing skill from Day 3's Cursor exercise, now doing real work.

---

## Part 3 — Create your CRM sheet (~10 min)
1. Go to Google Sheets → new blank sheet → name it `Lead CRM`
2. In **row 1**, add these exact column headers (order matters for easy mapping):

   | timestamp | lead_name | lead_company | tier | reasoning | suggested_opener | enrichment_used | hn_summary |
   |---|---|---|---|---|---|---|---|

3. Leave the rest blank. That's your database for now.

---

## Part 4 — Add + connect the Google Sheets node (~30 min)

1. In n8n, click the **+** after the **Parse Claude JSON** node
2. Search **Google Sheets** → select it
3. Set **Operation** to **Append Row** (or "Append or Update Row")
4. **Credential:** click *Create New Credential*
   - On n8n Cloud: choose **Google Sheets OAuth2** → click **Sign in with Google** → pick your account → allow access. (n8n Cloud handles the OAuth for you.)
   - *(Self-hosted n8n needs a Google Cloud Console setup — see docs.n8n.io if you went that route.)*
5. **Document:** select your `Lead CRM` sheet from the dropdown
6. **Sheet:** select `Sheet1`
7. **Mapping:** choose **Map Each Column Manually**. For each column, set the value to the matching field using an expression, e.g.:
   - `timestamp` → `{{ $json.timestamp }}`
   - `lead_name` → `{{ $json.lead_name }}`
   - `tier` → `{{ $json.tier }}`
   - ...and so on for each column

   (If the field names match the headers, n8n's **Map Automatically** option can do this in one click — try that first.)

---

## Part 5 — Run the whole thing end to end (~20 min)
1. **Execute workflow**
2. Open your Google Sheet — a new row should appear with the timestamp, lead, tier, reasoning, and drafted opener
3. Change the **Sample Lead** (try `Notion` / `notion.so`, or a student lead to force Tier C) and run again
4. Watch your CRM fill up, one qualified lead per row

🎉 **This is a complete agent loop:** input → enrich → reason → structured output → logged to a system of record. That's the shape of nearly every production agentic workflow.

---

## Part 6 — Architecture thinking (~30 min, the mindset shift)
Open `PROJECT-1.md`. The Mermaid diagram there renders automatically on GitHub — it's your README's centerpiece.

Then do this yourself: on paper or in Excalidraw (excalidraw.com, free), **redraw the workflow from memory.** Force yourself to remember:
- Where does data enter?
- Which steps can fail, and what happens when they do?
- Where's the human checkpoint?

Being able to whiteboard an agent's architecture — not just click nodes — is what interviewers actually probe for. "Walk me through a workflow you built" is the question. This diagram is your answer.

**Notice the HITL gate** in the diagram (the hexagon). It's not built yet — right now the agent logs everything with no human check. That's tomorrow's job, and it's the single most important production concept: the agent drafts, a human approves anything customer-facing.

---

## Ship (~15 min)
1. Download the workflow → save as `workflows/project-1-lead-qualifier.json`
2. Upload it + `PROJECT-1.md` + `DAY-4-SETUP.md` to GitHub
3. Commit: `Day 4: Project 1 happy path + Google Sheets CRM output`

---

## What you learned today
- Parsing an LLM's JSON string answer into clean, usable fields
- Connecting an OAuth service (Google Sheets) and mapping data to columns
- Completing a full agent loop: input → enrich → reason → log to a system of record
- Reading and drawing an architecture — the skill interviewers test with "walk me through what you built"

## Watch-outs
- **Claude output won't parse?** Open the Claude node's raw output. If it added extra text around the JSON, tighten the SOP's "return ONLY JSON" line. (Real validation comes Day 5.)
- **Google auth fails on n8n Cloud?** Make sure you complete the full "Sign in with Google" popup — it sometimes opens behind the main window.
- **Column mapping empty?** Check the header names in your sheet exactly match the field names, or map each one manually.

## Tomorrow (Day 5)
The HITL approval gate + guardrails: route Tier A leads to Slack/email for human approval before anything sends, and add validation so bad output never reaches your CRM.
