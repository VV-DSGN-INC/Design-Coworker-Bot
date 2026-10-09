# Design-Coworker-Bot

A planned Claude Code agent project that turns design conversations (meetings, Slack threads, in-person notes) into shared team summaries.

**Status:** pre-implementation. The repo holds the approved design spec only. There is no `CLAUDE.md`, skill, template or config yet, so there is nothing to run.

## What it will do

Opening the folder in Claude Code launches the bot, and a single `/summarize` command is the entry point. Per the spec:

- **Sources:** Granola meeting transcripts, Slack threads (by `slack.com/archives/` URL) and pasted notes.
- **Output:** a TL;DR, decisions, action items, open questions and links. Conversations that critique specific artifacts (screens, flows, Figma links) use a design-review template, anything else uses a general one.
- **Delivery:** Slack message, Slack canvas or Notion page, chosen per summary, plus a local markdown archive.
- **Guardrail:** nothing is posted to Slack or Notion until the final text is explicitly approved.
- **Access:** through already-connected MCP servers (Granola, Slack, Notion). The repo holds no credentials and no server code.

## Planned layout

```
CLAUDE.md                     bot behavior and conventions
config.md                     default Slack channel, Notion parent, language, test target
CHECKLIST.md                  source x delivery verification matrix
.claude/skills/summarize/     the /summarize skill
templates/                    design-review.md, general.md
summaries/                    local archive (gitignored, the repo is public)
```

## Docs

- [Design spec](docs/superpowers/specs/2026-06-10-design-coworker-bot-design.md)
