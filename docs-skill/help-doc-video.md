# Help-doc videos with Tella Auto Capture

How an agent makes the walkthrough video for a help article, with no one recording it. Make every help video this way so they all look alike. Every call here is a tool on the Tella MCP server (`https://api.tella.com/mcp`). Auto Capture needs a paid Tella plan and spends the workspace's AI credits for each take.

Fill in the `{{placeholders}}` once and commit this file next to your docs skill (`SKILL.md`), which links to it.

## One-time setup

- **A demo account in your product** (`{{DEMO_ACCOUNT_EMAIL}}`) that sees exactly what a customer sees. Never film with a staff or admin account: it sees feature flags, internal tools and staff-only menus. If your flag system targets by email domain, make sure the demo account's domain gets no internal targeting.
- **Demo data** for it: a demo project, document or record with realistic content that isn't about your own product (a recording of your product inside your product is confusing to watch). Pages that show members, billing or company settings get a **separate demo workspace/org** where the demo account is the only member, so no real names or emails appear on screen.
- **Its login saved in Tella** under **Settings → Credentials** (typed in or picked from 1Password), named `{{CREDENTIAL_NAME}}`. Auto Capture signs in off camera, so the clip starts signed in, and the password is never shown to the AI and is masked in recordings. `check_auto_capture_logins` confirms which saved login matches your app's URL.
- **One house style**, written down once (ideally as a Tella Agent workspace skill named `{{STYLE_SKILL}}`): zooms on clicks, step labels, playback speed, a "Read the guide" button, and anything else you want on every video. All styling comes from it, so change the look there, not per video.
- **A Tella playlist** for help videos (`{{PLAYLIST_ID}}`) and, optionally, one for archived ones.

## When to make one

Any change to the UI a video films gets a new video. It's that simple.

- **A new feature** in the app gets a video in its article.
- **Anything changed on a surface an article's video films** gets that video re-recorded: a new option, row, button, menu item or shortcut; a renamed, moved, restyled or removed control; a step added or removed. If the article has no video yet, make one.
- Don't decide whether the old video is still close enough. If the change alters what the filmed surface shows, re-record. None of these is a reason to skip: "the new option only appears under another setting", "it's display-only", "the old path still works", "it's hard to film". Find demo data or a setup that shows the change.
- Make the video in the same run that writes the article. Don't leave it for a person to ask for.
- Skip only these, and name the reason in the PR:
  - changes to text the video doesn't show, and backend-only changes;
  - features behind a flag customers don't have;
  - articles a browser recording can't show: billing, account security, troubleshooting, FAQ answers without steps, desktop or mobile apps, flows that need a connected third-party account.
- An article that describes several distinct tasks (add a thing, then edit or remove it) gets one video per task, each under its own section heading.

## No flagged or private UI, anywhere in the frame

- Before filming, find the UI labels the article names in source and look for a feature-flag check around them. If the article's own controls are flagged off for customers, don't film it until the flag ships.
- If the demo account sees some flagged UI elsewhere on the page, keep it out of frame with a zoom over the part you need, or fix the demo account's targeting.
- After delivery, look at every second of the take. A flagged panel, menu item, button or label anywhere on screen, or any real customer or staff data, means re-record, never blur. Blur only what is unavoidable and harmless, such as the demo account's own email in a sharing dialog.

## 0. Wait until the change is live

Auto Capture films your production app, and a docs-sync run starts minutes after the merge, before the change is deployed. Filming then records the old UI. Wait for the deploy yourself; never stop to ask anyone whether it has shipped.

1. Find what tells you production includes the merge commit: {{DEPLOY_CHECK, e.g. "the Deploy workflow run for the merge commit: `gh api \"repos/OWNER/REPO/actions/workflows/deploy.yml/runs?head_sha=<sha>\"`, then a version endpoint or deploy tag that contains the commit"}}. Check that production serves the commit, not only that a job finished: a later deploy can cancel or replace the one you watched and ship your change anyway.
2. Wait in bounded steps (your agent platform's wait or timer tool, or `timeout 540 bash -c 'until <check>; do sleep 30; done'`), never an open-ended `sleep`.
3. Stop after about {{DEPLOY_TIMEOUT, e.g. 90 minutes}} from the merge. Then leave the docs PR without the video and say in its body that the change wasn't deployed yet, with a link to the deploy run.

Open the docs PR before you wait, so the text doesn't wait on the video. In the take's instructions, name a label or line of copy that only the new version has and tell it to stop if it sees the old one.

## 1. Record the take

Call `create_auto_capture` with:

- `targetUrl`: a stable start page in your app (`{{APP_START_URL}}`, e.g. your dashboard), not a deep link. The take gets to the feature off camera, and that lead-in is cut.
- `instructions`, covering:
  - What the clip is for: "A short how-to clip for the help article \"<title>\", watched by <product> users."
  - Which login to use (`{{CREDENTIAL_NAME}}`) and which demo data to open, off camera, so the clip starts where the article's steps start.
  - The article's numbered steps, in its own words, and that they must be followed in order. End on the result: show the finished action and the feature's options, not just how to open it (save instead of cancelling; step through each choice the article lists).
  - The label only the new version has, and "stop if you see the old one".
  - Cleanup: undo everything the take changed after the visible steps, deleting anything it added rather than relying on undo. That cleanup is cut from the video.
  - For anything hard to undo, film on a copy: duplicate the demo data off camera, film on the copy, delete only the copy in the cleanup. Never open a panel or setting on the original.
  - "Keep everything in ONE browser tab. Close any exploration tab before filming, so no second live cursor appears in the clip."

Save the `sourceId` the moment `create_auto_capture` returns. A take runs 15 to 30 minutes: check it with `get_auto_capture` every 5 to 10 minutes using a timer, never a bare `sleep`. Run one take at a time per demo account: two takes in the same app show up in each other's footage.

## 2. Check the take

Read the handoff, but don't trust it: a handoff has claimed success while every frame showed an error. Look at frames across the whole take (`get_source_thumbnail`, about one per second) for:

- an error card, a blank or loading screen, or the old version of the UI;
- a second cursor or a collaborator name tag;
- flagged, staff-only or private UI, or real customer or staff data;
- steps that don't match the article. That usually means the article is wrong: fix the text in the same PR and say so.

Then check the demo data is as it was before the take. If the take changed it, restore it (a short maintenance take that undoes exactly that change works).

A take that ends needing guidance gets one `revise_auto_capture` with an answer. After two unusable takes, skip the video and say why in the PR.

## 3. Build and style the video

1. `create_video` with the `sourceId` and the article's title as its name, and nothing else.
2. Apply your house style (`{{STYLE_SKILL}}`) and nothing else: ask Tella Agent in the editor to run the skill, or follow its text exactly through the Tella MCP.
3. Cut the lead-in to about 0.5 s before the first visible click, and end before the pointer heads to the cleanup.
4. Time step labels and highlights from the frames, not the take's chapters, which can lead the clicks by about a second. A highlight must show before the click it marks.
5. Pick a speed the viewer can follow: around 1.7x for slow demos, 1.2x when steps are under about 3 s apart.
6. Check the result with `get_video_frame` at each step and at the last frame.

## 4. Publish

- Embed the video in the article right after the first paragraph (never before the answer):

  ```html
  <iframe
    src="https://www.tella.tv/video/VIDEO_ID/embed?b=0&title=0&a=1&loop=0&t=0&muted=0&wt=0"
    width="100%"
    style="aspect-ratio: 16/9; border: none; border-radius: 8px"
    allow="autoplay; fullscreen"
    allowfullscreen
  ></iframe>
  ```

- `update_video` with a call to action, label "Read the guide", linking to the article's public URL, so viewers of the share page can get back to the article.
- `add_video_to_playlist` to your help video playlist.
- **Re-recording an existing video:** add the new take as a clip in the existing video, style it, then delete the old clips. The video's ID, link, embed and playlist entry stay the same, and nothing is left to clean up.
- **Never delete a replaced video.** Archive it instead: rename it with an "[Archived] " prefix, give it a "Watch the new video" button linking to the replacement, move it to an archive playlist, and swap every embed of it for the new one. Leave widely shared videos (launch or announcement videos, changelog embeds) completely untouched: only swap the article's embed.
- In the docs PR body, link each video's view page and say what you checked.
