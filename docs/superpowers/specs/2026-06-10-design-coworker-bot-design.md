# Design Coworker Bot — Design Spec

**Date:** 2026-06-10
**Status:** Approved design, pre-implementation
**Repo:** https://github.com/weeeha/Design-Coworker-Bot (public)

## Problem

Nick (designer) has frequent conversations with coworkers — design reviews, critiques, planning syncs — across multiple channels: recorded meetings, Slack threads, and in-person chats. The decisions, feedback, and rationale from these conversations get lost. The team needs shared, readable summaries of what was discussed and decided, without Nick writing them by hand.

## What this is

A **Claude Code agent project** — not a deployed app. Opening this folder in Claude Code launches the bot: the project `CLAUDE.md` defines its behavior and `/summarize` is the single entry-point command. All external access goes through Nick's already-connected MCP servers (Granola, Slack, Notion). The repo contains no credentials and no server code.

## Decisions from brainstorming

| Question | Decision |
|---|---|
| Input sources | Mix: Granola meeting transcripts, Slack threads, pasted notes (in-person fallback) |
| Audience | The team — summaries are shared artifacts |
| Platform | Claude Code agent (skills in this repo, zero infrastructure) |
| Delivery targets | Slack message, Slack canvas, Notion page — chosen per summary; markdown archive always |
| Content focus | Design-aware: works on any conversation, adapts structure for design reviews |
| Approach | One `/summarize` skill (approach A); suite/routine deferred to future phases |

## Repo layout

```
Design-Coworker-Bot/
├── .gitignore                   # excludes summaries/ (public repo)
├── CLAUDE.md                    # bot behavior + conventions (persona, hard rules, pointers)
├── README.md                    # what it is, how to use it
├── config.md                    # defaults: Slack channel, Notion parent page, language, test target
├── CHECKLIST.md                 # verification checklist (source types × delivery targets)
├── .claude/skills/summarize/
│   └── SKILL.md                 # the /summarize skill — the bot's brain
├── templates/
│   ├── design-review.md         # summary template for critiques/reviews
│   └── general.md               # summary template for any other conversation
└── summaries/                   # local archive, one .md per summary (GITIGNORED)
```

**Public-repo constraint:** the GitHub repo is public, so `summaries/` is gitignored. The bot's machinery (skill, templates, config structure) is public; the archive of team conversations stays local. If the repo is later made private, removing the gitignore line turns the archive into a pushable team history. `config.md` is committed but must contain no sensitive values beyond channel/page names; if Nick considers those sensitive, config moves to a gitignored `config.local.md` (decide at first-run setup).

## Components

### `/summarize` skill
Orchestrates the full flow (below). Stateless between runs; all state lives in the archive files.

**Source detection rules:**
- Argument contains `slack.com/archives/` → Slack thread (read via Slack MCP)
- Argument like "last meeting", "meeting with X", or a meeting title → Granola transcript (query via Granola MCP)
- Multi-line pasted text → treated directly as raw notes (in-person fallback)
- Ambiguous or empty → list recent Granola meetings and ask Nick to pick

**Classification rule:** if the conversation critiques specific artifacts (screens, flows, components, Figma links present), use `design-review` template; otherwise `general`. The bot states which template it chose; Nick can override.

### Templates

`templates/design-review.md`:
- **TL;DR** — 2–3 sentences readable in 10 seconds
- **What was reviewed** — per screen/flow/component: feedback given, what was decided, and the rationale
- **Decisions** — consolidated list
- **Action items** — owner + due date only when actually mentioned, never invented
- **Open questions** — explicitly unresolved items
- **Links** — Figma files and anything else shared

`templates/general.md`: TL;DR, topics discussed, decisions, action items, open questions, links.

Style: English by default (configurable in `config.md`), neutral team-readable tone. Opinions attributed to named people only when the transcript is unambiguous.

### `config.md`
Created during first-run setup (the skill detects missing values and asks):
- `default_slack_channel` — where team summaries usually go
- `notion_parent` — parent page for Notion summaries
- `language` — default `en`
- `test_target` — Nick's own Slack DM, used for first-run verification

### Archive format
`summaries/YYYY-MM-DD-<kebab-topic>.md` with YAML frontmatter:

```yaml
---
date: 2026-06-10
source: granola | slack-thread | pasted-notes
source_ref: <meeting id, thread URL, or "manual">
type: design-review | general
participants: [name, name]
delivered_to: ["slack:#channel", "canvas:#channel", "notion:<page title>"]
---
```

`delivered_to` is updated after each successful delivery, making re-delivery and future `/deliver` tooling possible. `participants` is omitted when unknown (e.g. pasted notes) — never guessed.

## Data flow — one run

1. **Source** — Nick invokes `/summarize` with a meeting reference, Slack thread URL, or pasted notes.
2. **Fetch** — transcript via Granola MCP / thread via Slack MCP / pasted text as-is.
3. **Draft** — classify conversation, fill the matching template.
4. **Review** — draft shown to Nick in the session; iterate until approved. **Nothing is delivered before explicit approval of the final text.**
5. **Deliver** — Nick picks target(s); defaults from `config.md`. Post via Slack/Notion MCPs.
6. **Archive** — write the markdown file with frontmatter; record delivery targets.

`dry-run` modifier: steps 1–4 and 6 only — no delivery.

## Guardrails & error handling

- **Hard rule (non-negotiable, written into the skill):** nothing is posted to Slack or Notion without Nick's explicit approval of the final text.
- Meeting not found → list recent Granola meetings; never guess.
- Slack thread unreadable (private channel, missing scope) → state it plainly; offer the paste fallback.
- Delivery failure → archive copy already exists; report which target failed; retry only on request.
- Unclear transcript (cross-talk, ambiguous speakers) → mark uncertainty in the draft (e.g. "unclear who raised this") rather than misattributing.

## Testing / verification

Behavioral verification, no unit tests:
- **Dry-run mode** for safe iteration on real sources.
- **First-run check:** full flow once per source type (one real Granola meeting, one real Slack thread, one pasted note), delivering only to `test_target` (Nick's DM) before any team channel is touched.
- **`CHECKLIST.md`:** a two-minute re-verification matrix (each source type × each delivery target) to run after any future edit to the skill.

## Out of scope (future phases)

- `/deliver` (re-post an archived summary) and `/recent` (browse sources) as separate skills — approach B; the frontmatter `delivered_to` field is the seam that enables them.
- Scheduled end-of-day routine that auto-drafts summaries of new meetings — approach C.
- Live recording of in-person conversations (covered today by Granola's mic recording or pasted notes).
- Making the repo private + pushing the archive as team history.
