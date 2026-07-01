# Day 3 — Live Enrichment + Directing Cursor to Write Code

Yesterday your agent qualified a hardcoded lead. Today it pulls **real data from the internet** before making a decision — and you start using Cursor to write the tricky parts for you. This is the "engineer hands" skill: knowing what to ask, not memorising syntax.

**What's new in v3**
- A **HackerNews API** call enriches each lead with real company mentions before Claude qualifies them (free, no API key needed — perfect for learning the pattern)
- A **fallback branch** handles enrichment failure gracefully — Claude still runs even if the API is down
- The **Final Output** node bundles everything: Claude's verdict + the enrichment data that informed it
- The prompt now includes enrichment context so Claude's reasoning is grounded in real signal

---

## Part 1 — Import v3 (~10 min)
1. n8n → new workflow → **Import from File** → `workflows/lead-qualifier-v3.json`
2. Re-select your **Anthropic credential** on the *Claude - Qualify Lead* node
3. No other credentials needed — HackerNews API is public

The chain is now:
```
Start → Skill Library → Sample Lead → HackerNews Enrichment
                                            ↓ success        ↓ fail
                                       Parse Enrichment   Enrichment Fallback
                                            └──────┬────────┘
                                                   ↓
                                          Claude - Qualify Lead
                                            ↓ success    ↓ fail
                                        Final Output   Handle Error
```

---

## Part 2 — Understand the new nodes (~20 min)

**HackerNews Enrichment node**
Open it. It's a plain HTTP GET request — no auth needed:
```
https://hn.algolia.com/api/v1/search?query=stripe.com&tags=story&hitsPerPage=3
```
This searches HackerNews for the company domain and returns the top 3 stories. That's it. Any public API follows this exact same pattern: URL + optional params + no/some auth.

**Parse Enrichment node**
Open it and look at the `hn_summary` expression:
```
={{ $json.hits && $json.hits.length > 0
    ? $json.hits.map(h => '- ' + h.title + ' (' + h.points + ' points)').join('\n')
    : 'No HackerNews mentions found.' }}
```
This is the most complex expression you've seen so far. Don't memorise it — **this is exactly the kind of thing you ask Cursor to write.** See Part 4.

**The two-branch merge**
Both Parse Enrichment and Enrichment Fallback connect into Claude. n8n runs whichever branch succeeded — Claude always gets *something*, even if enrichment failed. This is the fallback pattern you'll use in every production workflow.

---

## Part 3 — Run it and read the output (~20 min)

1. Execute the workflow
2. Open **Final Output** — you should see:
   - `claude_raw` → the JSON verdict (tier / reasoning / opener / enrichment_used)
   - `lead_company` → Stripe
   - `hn_summary` → real HackerNews story titles about Stripe

3. **Now change the company domain.** Open **Sample Lead** and try:
   - `lead_company: "Notion"`, `company_domain: "notion.so"`
   - `lead_company: "Acme Corp"`, `company_domain: "acmecorp-fake-domain.xyz"` (tests the fallback)

   Re-run each time. Watch how Claude's reasoning changes when enrichment finds real signal vs nothing.

4. **This is the core insight of Day 3:** the agent's decision quality improves with better context. The enrichment step is just context-building. In Week 2 you'll replace HackerNews with your own Supabase knowledge base — same pattern, much richer data.

---

## Part 4 — Directing Cursor to write code (~60 min, the main skill today)

Cursor is an AI-powered code editor. You describe what you want in plain English — it writes the code. Your job is to describe clearly and verify the output makes sense. You never need to memorise syntax.

**Setup (5 min)**
- Open Cursor (download at cursor.com if not done)
- Open a new file, save it as `expressions-practice.js`

**Exercise 1 — Rewrite the Parse Enrichment expression**
Type this prompt into Cursor's chat (Cmd+L):

> "I have a HackerNews API response in a variable called `hits`. It's an array of objects, each with a `title` and `points` field. Write a JavaScript one-liner that maps over the array and returns a string where each item is formatted as `- Title (X points)` on its own line. If the array is empty or undefined, return the string `No HackerNews mentions found.`"

Cursor writes it. Read it. Does it match what's in the Parse Enrichment node? It should be nearly identical. **That's the point — you just learned to generate that expression by describing the problem, not memorising JS.**

**Exercise 2 — Parse Claude's JSON output**
Claude returns its answer as a raw string like:
```
{"tier":"A","reasoning":"...","suggested_opener":"...","enrichment_used":"..."}
```
You need to turn that string into a real object so you can use each field separately downstream. Ask Cursor:

> "In n8n I have a field called `claude_raw` that contains a JSON string. Write an n8n expression that parses it into an object and pulls out just the `tier` field. Handle the case where parsing fails by returning the string `PARSE_ERROR`."

Copy the expression Cursor gives you. In n8n, add a new **Set node** after Final Output and paste it in as an expression for a new field called `tier`. Run the workflow — you should see just `A`, `B`, or `C` as a clean field.

**This is the workflow for the rest of the sprint:**
1. Describe what you need in plain English to Cursor
2. Cursor writes the expression or code
3. You paste it into n8n and verify it works
4. If it breaks, paste the error back into Cursor and ask it to fix it

You are the architect. Cursor is the hands.

---

## Part 5 — Read one real API doc (~30 min)

The HackerNews API is the simplest possible API. Before Day 4 you should be able to read any API doc and answer: *what URL do I call, what params does it take, do I need auth, what does the response look like?*

Go to: `https://hn.algolia.com/api`

Find:
- What does the `search` endpoint return?
- What other `tags` values can you filter by besides `story`?
- What field contains the URL of the original article?

Then try changing the HackerNews node URL to also return `comment` type results. Run it. See how the response changes.

---

## Ship (~15 min)
1. Download the v3 workflow from n8n → save as `workflows/lead-qualifier-v3.json`
2. Upload to GitHub `workflows/` folder
3. Also upload `DAY-3-SETUP.md`
4. Commit: `Day 3: live enrichment, fallback branch, Cursor for expressions`

---

## What you learned today
- The standard API call pattern: URL + params + parse response (reusable for any API forever)
- Fallback branches: agents degrade gracefully, never crash on bad data
- Context quality drives decision quality — enrichment makes Claude smarter
- Directing Cursor to write expressions: describe the problem, verify the output, paste it in

## Watch-outs
- If HackerNews returns no hits for a domain, that's fine — the fallback text still flows to Claude
- The `enrichment_used` field in Claude's output tells you whether it actually used the data — check it
- Cursor sometimes over-engineers simple expressions. If the output looks complex, ask: *"can you simplify this to a one-liner?"*

## Tomorrow (Day 4)
Design Project 1 end-to-end: architecture diagram, the full happy-path build, and wiring a real output destination (Google Sheets as a simple CRM log).
