---
name: "docs"
description: "Create or update {{PRODUCT}}'s help center and developer docs: articles under {{DOCS_DIR}} plus the navigation in {{NAV_FILE}}. Use when asked to write, add, update, or reorganise a docs article, from any source: a request, a support ticket, a merged PR, an automation."
---

Write and update docs in `{{DOCS_DIR}}`.

- Articles: `{{ARTICLES_DIR, e.g. docs/help/}}`
- Navigation: `{{NAV_FILE, e.g. docs/docs.json, mkdocs.yml, sidebars.js}}`
- Redirects: `{{REDIRECTS_LOCATION}}`
- Live at: `{{DOCS_URL}}`

## The rule that governs everything: the sidebar mirrors the product

Group the sidebar by **the surface a feature lives on in the app**: a toolbar button, a settings page, a dialog, a CLI subcommand. A reader who just used a feature should find its article under the name of the thing they clicked.

Two consequences that catch people out:

**The folder is not the taxonomy.** `<articles>/<folder>/<slug>` only decides the public URL. A page in `troubleshooting/` can sit in the Record group, and a page in `faq/` can sit under Edit. Folder and group disagreeing is normal and correct. Don't "fix" it, and never pick a group because of the folder a file lives in.

**Never invent a group name.** Use the label the product uses. Groups named after things the UI has never said ("Video Effects", "AI Features", "Advanced") are how a docs tree becomes unusable.

## Before you touch the navigation

Re-derive the surfaces from source every time. The product moves and any list written down here goes stale; these commands don't:

```bash
{{COMMANDS_THAT_LIST_PRODUCT_SURFACES, e.g.
grep -n 'label="' src/components/Toolbar.tsx        # toolbar buttons, in rendered order
grep -rn 'title:' src/routes/settings/              # settings pages
}}
```

Code names lag the UI (a component called `Trim` whose button says "Cut"). The rendered label is the word the reader saw, so the label names the group.

Place the article under the group named after the button or page it documents. If the feature has no surface of its own (a workflow recipe, a symptom, an app-wide mechanic), put it in the closest task group: {{TASK_GROUPS, e.g. "Getting started", "Basics", "Troubleshooting"}}.

**If the product renamed or merged a tool, rename or split the group in the same PR.** This is the failure mode to watch for: a tool gets renamed, new articles keep getting appended to the group named after the old tool, and within a few PRs the sidebar describes an app that no longer exists.

## Creating an article

1. Pick the **group** first, from the product surface. That's the decision that matters.
2. Pick the **folder** second, and boringly: the closest existing one. Don't create a folder to express a category; folders are URLs.
3. Create `{{ARTICLES_DIR}}<folder>/<kebab-case-name>.{{EXT, e.g. mdx}}` with the required frontmatter:
   ```yaml
   ---
   title: "Human-readable title"
   description: "The answer, in one sentence a search result can show."
   ---
   ```
4. Add it to the navigation in that group, next to the articles it belongs beside.
5. Run the checks (below).

## Updating an article

Edit the file in place. No navigation change is needed.

**Never move or rename an existing article.** Its path is a public URL with search rankings, inbound links, in-app deep links, redirects, and possibly an AI search index behind it. Regrouping in the navigation costs nothing and gets readers to the same place. If a move is truly unavoidable, add a redirect from the old path and repoint any existing redirect that targeted it, so you don't create a chain.

## Style

Every article is read three ways: by a person skimming for their answer, by retrieval in an AI support bot (which embeds and returns single sections), and by answer engines (Google AI Overviews, ChatGPT, Perplexity) that quote the first self-contained answer they find. The same shape serves all three.

- **The first sentence is the answer.** For a yes/no question, that's "Yes." or "No." plus one clause. For a feature or task, it's the one sentence that says what it does and where it lives. Nothing comes before it: no video, no "This guide will explain…". Embeds and screenshots go after the first paragraph.
- **The `description` is that same answer**, not a description of the page. It's the search snippet, the `llms.txt` summary, and what support bots show. "Learn how to…" and "How to…" are wrong.
- **Titles: imperative for tasks, question for facts.** "Blur your background" for something you do, "Is there a free plan?" for something you want to know. The title is what the sidebar shows, so it also names the feature the reader clicked.
- **Each `##` section answers one sub-question on its own**, with a heading people would actually type ("Change the blur amount", not "Next steps"). Put a limit or a "no" in the section where someone would look for it, not only in a note at the top.
- **Say plainly what the product doesn't do.** "Is there a mobile app?" answered with "No." and the nearest alternative gets cited. A page that never mentions it leaves the answer to a competitor.
- Write in second person, directly and concisely. Numbered lists for steps, bullets for options. What, then how, then why.
- **Quote UI labels exactly as they render, checked against source**, not from memory and not from an older article that may itself be stale.
- {{FORMAT_GOTCHAS, e.g. "MDX: escape every `$` in a price as `\$`, or two on one line render as LaTeX."}}

## Walkthrough videos

Make an article's video with Tella Auto Capture, following [help-doc-video.md](help-doc-video.md). A new feature gets a video, and any change to a surface an existing video shows gets it re-recorded. Wait for the change to deploy, then record it in the same run.

## Check your work

```bash
{{DOCS_CHECK_COMMANDS, e.g.
npx mint broken-links
npm run docs:lint
}}
```

Worth automating in CI if you haven't: every article in exactly one group, every navigation entry pointing to a file, group sizes within bounds (say 2 to 10), required frontmatter present, no dead internal links. Those catch the mechanical decay. Whether a group still matches a real product surface is a judgement the checks can't make; that's yours.

## Handoff

No docs change ships silently, wherever it came from.

1. Open a PR and request `{{DOCS_OWNER}}` as a reviewer, in addition to anyone else the work needs. If the reviewer doesn't stick at create time, add it with `gh api -X POST repos/{{OWNER/REPO}}/pulls/<n>/requested_reviewers -f "reviewers[]={{DOCS_OWNER}}"`.
2. Post the PR link to `{{DOCS_CHANNEL}}` with one line on what changed and why.
