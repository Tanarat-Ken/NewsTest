# Daily Business Research — Fabric & Yarn Procurement Skill

A Claude Remote Routine skill that publishes a daily procurement intelligence brief about
fabric, yarn, raw material prices, logistics costs, and supply-chain news — targeting
**corporate procurement teams** in the Thai textile industry.

---

## What It Does

Each run (no Bash, no shell, no git CLI — pure Claude + MCP connectors):

1. **Sweeps 24-hour news** across Thai and international sources (WebSearch + WebFetch)
2. **Deduplicates** against past articles — a story is re-published only if its impact score
   increases by 2+ points on a 1–10 scale
3. **Drafts** an article with full inline citations — no fabricated statistics
4. **Analyzes** through three lenses: CEO / Corporate Strategist / Competitor
5. **Rewrites** into a cohesive brief (800–1,200 words Thai) with an `## Action Items` section
6. **Commits** `articles/YYYY-MM-DD-<slug>.md` to `main` via the GitHub MCP connector
7. **Notifies** a LINE channel with headline + 3-bullet TL;DR + permalink (optional)

---

## Repository Structure

```
.claude/
  skills/
    daily-business-research/
      skill.md          ← Full skill definition (phases 0–8, guardrails)
reference/
  sources.md            ← Pre-vetted source list with trust tiers
  perspectives.md       ← CEO / Strategist / Competitor framework + impact rubric
articles/
  YYYY-MM-DD-<slug>.md  ← Published briefs (committed by the skill)
.env.example            ← Required environment variables
.gitignore
README.md
```

---

## Running in Claude Web Routine

### Prerequisites

| Requirement | Details |
|---|---|
| Claude plan | Claude Pro or Team (Remote Routines feature) |
| GitHub connector | Must be connected and authorized in Claude settings |
| Repository | This repo, with `main` branch writable by the connector |
| LINE Bot (optional) | LINE Messaging API channel with push permission |

### Step 1 — Connect GitHub

1. Go to **Claude.ai → Settings → Connectors** (or the Routines panel).
2. Add the **GitHub** connector and authorize it to the repository
   `<your-org>/<this-repo>`.
3. Confirm the connector can read and write to `main`.

### Step 2 — Set Environment Variables

In the Claude Routine environment configuration, add:

| Variable | Required | Description |
|---|---|---|
| `GITHUB_OWNER` | Yes | GitHub username or organization |
| `GITHUB_REPO` | Yes | Repository name |
| `LINE_CHANNEL_ACCESS_TOKEN` | No | LINE bot long-lived access token |
| `LINE_TO` | No | LINE User ID (`Uxxxxxxx`) or Group ID (`Cxxxxxxx`) |

> If `LINE_CHANNEL_ACCESS_TOKEN` or `LINE_TO` are absent, the skill skips the
> LINE notification step and commits the article normally.

### Step 3 — Create the Routine

1. In Claude Web, create a new **Routine** (or Remote Routine).
2. Set the **skill path**: `.claude/skills/daily-business-research/skill.md`
3. Set the **schedule**: daily at your preferred time, e.g. `0 7 * * *` (07:00 Asia/Bangkok = 00:00 UTC).
4. Enable the routine.

### Step 4 — First Run

Trigger a manual run from the Routine panel to verify:

- GitHub connector responds (Phase 0 preflight)
- At least one article is produced and committed to `articles/`
- If LINE is configured, a test notification is received

Check the run log output (Phase 8) for any errors.

---

## Guardrails

| Situation | Behaviour |
|---|---|
| GitHub connector unavailable | Abort immediately, log `[ABORT]` |
| LINE env vars not set | Skip LINE step, commit proceeds normally |
| LINE API returns non-200 | Log `[ERROR]`, no retry, continue |
| Duplicate story, no impact uplift | Skip story silently |
| No new stories today | Commit a quiet-day placeholder, skip LINE |
| Paywalled or fetch-failed URL | Discard and log `[SKIP]` |
| Fabricated data | Prohibited — skill must cite or omit |

---

## Article Format

Each committed file follows this structure:

```markdown
---
title: "..."
date: "YYYY-MM-DD"
topics_covered: [slug1, slug2]
impact_scores: {slug1: 7, slug2: 5}
sources_count: N
---

# Headline

> TL;DR — ...

## สรุปประเด็นสำคัญ
## รายละเอียดข่าว
## ผลกระทบต่อการจัดซื้อองค์กร
## Action Items
## Sources
```

---

## LINE Notification Format

```
📊 Daily Procurement Brief — YYYY-MM-DD

{HEADLINE}

สรุป:
• {bullet 1}
• {bullet 2}
• {bullet 3}

อ่านฉบับเต็ม: https://github.com/<owner>/<repo>/blob/<SHA>/articles/YYYY-MM-DD-<slug>.md
```

---

## Customizing Sources

Edit `reference/sources.md` to add or remove domains.
The skill's WebSearch queries are biased toward domains listed as **Tier 1** in that file.

## Customizing the Analysis Framework

Edit `reference/perspectives.md` to adjust the CEO / Strategist / Competitor question sets,
the impact score rubric, or the Action Items format.

---

## Local Development Note

This skill is designed for Claude Remote Routine (no shell access).
All steps use only `WebSearch`, `WebFetch`, and `mcp__github__*` tools.
**Do not** add Bash-dependent steps — they will fail in the remote sandbox.
