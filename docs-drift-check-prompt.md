# Docs drift check: a weekly prompt that audits your docs against the product

The docs-sync prompt catches changes as they merge. This one catches what slipped through: renamed controls, sidebar groups named after tools that no longer exist, shipped features with no article, and articles whose steps no longer work. Run it weekly on a schedule with a coding agent that has the repo checked out on a fresh branch, `gh`, and (optionally) the Slack and Tella MCP servers.

Fill in the `{{placeholders}}` and delete the optional sections you don't want.

---

You are auditing `{{PRODUCT}}`'s docs against the product they document. Docs live in `{{DOCS_DIR}}`, with navigation in `{{NAV_FILE}}`. You have the full repo checked out at the latest `{{DEFAULT_BRANCH}}`.

Read `{{DOCS_SKILL_PATH}}` first. It defines how the docs are organised, and every edit must follow it.

**Don't re-check what CI already checks** ({{CI_DOCS_CHECKS, e.g. "orphan pages, nav entries with no file, group sizes, missing frontmatter, dead internal links"}}). Your job is the part a test can't judge: whether the docs still describe the app as it is today.

## 1. Do the sidebar groups still name real product surfaces? (always)

Re-derive the surfaces from source, never from an older article, a written-down list, or this prompt:

```bash
{{COMMANDS_THAT_LIST_PRODUCT_SURFACES}}
```

Trust the rendered label, not the component name. Compare against the group names in the navigation and report:

- a group named after a surface the app no longer has;
- a surface the app has that no group corresponds to;
- one group holding pages from two distinct surfaces, or one surface split across two groups.

This is the highest-value check. A tool getting renamed while its group keeps the old name is exactly how a sidebar drifts.

## 2. Do articles quote UI labels that still exist?

Articles tell readers to click **bolded labels**. When the app renames a control, the article silently becomes wrong. You can't check every article in one run, so check:

- every article changed in the last 14 days (`git log --since='14 days ago' --name-only --pretty=format: -- {{DOCS_DIR}} | sort -u`);
- plus about 15 more, rotating: the ones least recently modified (`git log -1 --format='%ad' --date=short -- <file>`), so the whole set gets covered over time.

For each, take the bolded UI strings and check the exact string still appears in `{{UI_SOURCE_DIR}}`. Report only labels you can show are gone, with the article line and the source file that proves the current wording. For surfaces whose source isn't in this repo (a mobile or desktop app), say you couldn't verify them; don't guess.

## 3. Shipped to everyone, but undocumented

List the app's settings pages, toolbar items and other top-level surfaces, and find ones with no article. Before calling something a gap, confirm every customer has it: a feature-flag check in code doesn't mean unreleased (the flag may be fully rolled out), and a flag at 0% does. {{HOW_TO_CHECK_FLAG_ROLLOUT}}. Report a gap only when it's on for everyone; give the rollout fraction otherwise.

## 4. Documented, but not shipped

The reverse: articles describing a feature whose flag is off for customers. Report these. They need a product decision (keep, gate, or label), not an edit.

## 5. Do the steps still work? (optional, needs Tella Auto Capture)

Pick 3 articles with numbered steps from the rotation in step 2. For each, start a Tella Auto Capture take (`create_auto_capture` on the Tella MCP) on `{{APP_START_URL}}`, signed in with the `{{CREDENTIAL_NAME}}` demo login, with the instruction: "Follow the steps in this article exactly, in order: <steps>. Don't fix or work around anything. Report every step where the button, label or result in the app doesn't match what the article says." Run one take at a time and check each with a timer, not `sleep`.

Look at the take's frames yourself. A step the take couldn't follow, or a result that differs from the article, is drift: report it with the frame that shows it. If the article has a video and the app no longer looks like it, list it as needing a re-record (see `{{VIDEO_RECIPE_PATH}}`).

## What to do with what you find

Open **at most one PR**, and only for drift you can prove from source:

- renaming or splitting a sidebar group to match the current labels;
- correcting a UI label in an article where source shows the exact current string;
- fixing a step that a take in section 5 showed is wrong, when source confirms it.

Everything that needs judgement goes in the report as a finding, not a commit: coverage gaps, features to retire, articles that need rewriting, anything you couldn't verify. Don't rewrite articles wholesale, don't reorganise groups that are still accurate, and don't churn wording you merely dislike. A quiet week with no PR is a good outcome.

**Never move or rename an article file.** Its path is a public URL. Regroup in the navigation instead.

If you edit: run `{{DOCS_CHECK_COMMANDS}}`, stage only the files you changed (`git add <paths>`), push with `git push -u origin HEAD`, and open a PR titled "Docs drift: <one line>" whose body names what drifted and the source file that proves it, requesting `{{DOCS_OWNER}}` as reviewer. Never merge it.

## Report (always)

Post to `{{SLACK_CHANNEL_ID}}` every run, whether or not you opened a PR (use the channel ID directly; don't list channels). Mention `{{REVIEWER_SLACK_ID}}`. Keep it short:

- one line on whether the sidebar still matches the product, naming any group that doesn't;
- stale labels found, as `article -> old label -> current label`;
- coverage gaps, with the rollout evidence;
- steps that didn't work in a take, if you ran section 5;
- the PR link, or "no PR: nothing provable this week";
- how many articles you checked, so the rotation is visible.

Say plainly what you couldn't verify. Never present a guess as a finding.
