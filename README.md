# Docs agent kit

Prompts and a skill that keep a help center in step with a product that ships every day: the written articles and their walkthrough videos. They're the generic versions of what Tella runs on every pull request; the story is in [How I created self-updating help docs (with videos)](https://www.tella.com/blog/self-updating-help-docs-with-videos).

| File | What it is | How to run it |
| --- | --- | --- |
| `docs-sync-prompt.md` | Reads each merged PR, updates the docs it affects, records or re-records the walkthrough video, opens a docs PR | An agent triggered on every merged PR (a GitHub Action on `pull_request: closed` + `merged`, or your agent platform's "PR merged" event) |
| `docs-drift-check-prompt.md` | Weekly audit: sidebar groups vs the real UI, stale labels, undocumented and unshipped features, and (optionally) whether article steps still work | An agent on a weekly schedule |
| `docs-skill/SKILL.md` | How your docs are organised and written: sidebar-mirrors-the-product, no URL moves, answer-first style, checks, handoff | Commit to your repo (e.g. `.agents/skills/docs/`) so every agent that touches docs reads it |
| `docs-skill/help-doc-video.md` | Recipe for making help videos with Tella Auto Capture: when to film, demo account, waiting for the deploy, the take, frame checks, styling, publishing and archiving | Committed next to the skill; the sync prompt follows it |

## Setup

1. Copy `docs-skill/` into your repo and fill in its `{{placeholders}}`: docs paths, the commands that list your product's UI surfaces, your docs checks, and the reviewer.
2. Fill in the placeholders in the two prompts and set them up as automations. Delete the optional sections (Slack, videos, Auto Capture step checks) you don't want.
3. For videos: connect your agent to the Tella MCP server (`https://api.tella.com/mcp`), create a demo account in your product, save its login in Tella under **Settings → Credentials**, and write your house video style once (ideally as a Tella Agent workspace skill). Auto Capture needs a paid Tella plan and uses AI credits per take.

The agent never merges: every change arrives as a PR for a person to review.
