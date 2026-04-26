# Skill: Daily Business Research — Fabric & Yarn Procurement

> **Runtime**: Claude Remote Routine (Web)
> **Compatible tools**: WebSearch, WebFetch, mcp__github__* connectors
> **Prohibited**: Bash, shell commands, git CLI, any subprocess execution

---

## Purpose

Produce a daily procurement intelligence brief covering news from the **past 24 hours** about:

- Fabric and yarn prices (domestic Thailand + international)
- Raw material procurement, supply chain, logistics
- Oil/fuel prices and transport costs affecting textile supply chains
- Import/export policy, tariffs, FX movements impacting fabric/yarn trade

Audience: **Corporate procurement teams** at fabric/yarn buyers and manufacturers.

---

## Phase 0 — Preflight Checks

Before any research, verify connectors are available.

```
REQUIRED_CONNECTOR: mcp__github__
TIMEZONE: Asia/Bangkok (UTC+7)
TODAY: resolve from system date in Asia/Bangkok timezone
LINE_ENABLED: true (notifications sent via n8n webhook)
```

1. Attempt a lightweight GitHub API call (e.g., `mcp__github__get_me`) to confirm the GitHub connector is live.
   - **If it fails**: log `[ABORT] GitHub connector unavailable — cannot commit article. Halting skill.` and stop all further steps.
2. Set `LINE_ENABLED = true`. LINE notifications are sent via n8n webhook — no env vars required.

---

## Phase 1 — Deduplicate Against Past Articles

Retrieve the list of files already committed under `articles/` in the GitHub repo via `mcp__github__get_file_contents` or directory listing.

For each file found:
- Parse the filename (`YYYY-MM-DD-<slug>.md`) to determine its date.
- Read the file via `mcp__github__get_file_contents`.
- Extract the `## Topics Covered` front-matter section and store topic slugs for deduplication.

Build a **seen-topics registry** (in-memory list of `{slug, date, impact_score}` tuples).

**Deduplication rules (enforced in Phase 3):**
- A story with the same topic slug as a past article is **skipped** unless its computed `impact_score` exceeds the previous article's score by ≥ 2 points on the 1–10 scale.
- Impact score is derived during research (Phase 2) based on: price movement magnitude, supply disruption breadth, geographic spread, and number of corroborating sources.

---

## Phase 2 — Research (24-Hour News Sweep)

Run the following **WebSearch** queries. Use date filters where the engine supports them (`after:YYYY-MM-DD`). Collect up to **5 credible results per query**, capturing: `title`, `url`, `source_name`, `publish_date`, `snippet`.

### Query Set A — Thai domestic

```
"ผ้า" OR "เส้นด้าย" OR "สิ่งทอ" ราคา 2026 site:thansettakij.com OR site:prachachat.net OR site:bangkokbiznews.com
"จัดซื้อ" OR "procurement" ราคาวัตถุดิบ ผ้า เส้นด้าย ไทย 2026
ราคาน้ำมันดีเซล ขนส่ง โลจิสติกส์ ไทย 2026
```

### Query Set B — International (English)

```
yarn fabric price index 2026 supply chain
polyester cotton yarn price increase 2026
textile raw material procurement global 2026
oil fuel surcharge logistics textile 2026
China yarn export price 2026
Bangladesh India textile export 2026
```

### Query Set C — Financial/Macro signals

```
USD THB exchange rate impact textile import 2026
shipping container rate textile Asia 2026
OPEC oil production cut textile supply chain 2026
```

For each result collected:
1. Fetch the full article via `WebFetch` (read-only GET).
2. Extract: headline, date, key data points (prices, percentages, volumes), affected companies/countries, and a 2-sentence summary.
3. Assign an `impact_score` (1–10) based on magnitude and breadth.
4. If the URL returns non-200 or content is paywalled/empty: discard and log `[SKIP] <url> — fetch failed or paywalled`.

**After collection**, apply deduplication rules from Phase 1. Remove any story that duplicates a past article without sufficient impact score uplift.

If **zero unique stories** remain after deduplication: log `[INFO] No new stories above deduplication threshold today.` and commit a brief placeholder article noting the quiet news day, then skip LINE notification.

---

## Phase 3 — Draft Article

Compose the article with the following structure. Write in **Thai** unless a section is explicitly marked bilingual.

```markdown
---
title: "{HEADLINE}"
date: "YYYY-MM-DD"
topics_covered:
  - slug1
  - slug2
impact_scores:
  slug1: 7
  slug2: 5
sources_count: N
---

# {HEADLINE}

> **TL;DR** — {one sentence, max 40 words}

## สรุปประเด็นสำคัญ

{bullet list — one bullet per story, cite [Source Name](URL)}

## รายละเอียดข่าว

{For each story: paragraph with concrete data points, prices, dates, affected parties.
Every factual claim must be followed by an inline citation ([Source](URL)).
NO invented statistics. If data is not in source, say "ยังไม่มีข้อมูลยืนยัน".}

## ผลกระทบต่อการจัดซื้อองค์กร

{2–3 paragraphs on direct implications for corporate procurement teams.
Focus on: contract timing, buffer stock strategy, supplier diversification, FX hedging.}
```

**Citation rules:**
- Every factual statement requires an inline Markdown link to the source URL.
- Do not fabricate or infer prices/statistics not present in the fetched content.
- If two sources conflict on a number, present both with their respective citations.

---

## Phase 4 — Three-Perspective Analysis

Load the framework from `reference/perspectives.md` in the repo via `mcp__github__get_file_contents`.

Apply all three lenses to the article drafted in Phase 3:

| Perspective | Focus |
|---|---|
| CEO | Strategic risk, market positioning, board-level decisions |
| Corporate Strategist | Competitive moat, supplier relationships, scenario planning |
| Competitor | How rivals might exploit this news, threat vectors |

For each perspective, produce 3–5 bullet points tied to **specific stories** from Phase 3.

---

## Phase 5 — Final Rewrite

Merge the Phase 3 draft and Phase 4 analysis into a single coherent article. Rules:

1. Perspectives are **woven into the narrative**, not appended as separate sections.
2. Add a final `## Action Items` section:
   - 3–5 concrete, time-bound actions for procurement teams.
   - Format: `[ ] Action — Owner suggestion — Deadline/trigger`
3. Word target: **400–600 words** in Thai.
4. Maintain all citations from Phase 3.
5. Append a `## Sources` section listing every URL cited, in order of appearance.

Final filename: `articles/YYYY-MM-DD-<slug>.md` where `<slug>` is a 3–5 word kebab-case summary of the dominant topic.

---

## Phase 6 — Commit to GitHub

Use `mcp__github__create_or_update_file` with:

```json
{
  "owner": "<GITHUB_OWNER>",
  "repo": "<GITHUB_REPO>",
  "path": "articles/YYYY-MM-DD-<slug>.md",
  "message": "brief: {TOPIC} YYYY-MM-DD",
  "content": "<article content>",
  "branch": "main"
}
```

- Capture the **commit SHA** returned in the response.
- Construct the permalink:
  `https://github.com/<GITHUB_OWNER>/<GITHUB_REPO>/blob/<COMMIT_SHA>/articles/YYYY-MM-DD-<slug>.md`
- Log `[OK] Committed: <permalink>`.

If `mcp__github__create_or_update_file` returns an error:
- Log `[ERROR] GitHub commit failed: <error_message>`. Do not retry silently. Halt Phase 7.

---

## Phase 7 — LINE Notification

**Always execute** (LINE_ENABLED = true).

Compose a short message (max 300 characters total):

```
📊 {HEADLINE} — {DATE}
• {bullet 1 — max 20 words}
• {bullet 2 — max 20 words}
• {bullet 3 — max 20 words}
อ่านเพิ่ม: {permalink}
```

URL-encode the message, then send via `WebFetch` (GET):

```
URL: https://n8n.srv1307565.hstgr.cloud/webhook/line-notify?msg={URL-encoded message}
```

- If WebFetch returns a response containing `"status":"ok"`: log `[OK] LINE notification sent.`
- If the response does not contain `"status":"ok"` or fetch fails: log `[ERROR] LINE webhook failed`. **Do not retry.** Continue to Phase 8.

---

## Phase 8 — Run Log Summary

Print to output:

```
=== Daily Business Research Run Log ===
Date        : YYYY-MM-DD (Asia/Bangkok)
Stories raw : N
After dedup : M
Article     : articles/YYYY-MM-DD-<slug>.md
Commit SHA  : <sha>
Permalink   : <permalink>
LINE sent   : yes | no
Errors      : none | <list>
========================================
```

---

## Guardrails Summary

| Rule | Behaviour |
|---|---|
| No Bash/shell/git CLI | Never invoke subprocess, terminal, or CLI commands |
| GitHub connector absent | Abort immediately, log error |
| LINE webhook fail | Log error, no silent retry, continue to Phase 8 |
| Duplicate story | Skip unless impact_score uplift ≥ 2 |
| No stories today | Commit quiet-day placeholder, skip LINE |
| Fabricated data | Prohibited — cite or omit |
| Paywalled source | Discard and log |
