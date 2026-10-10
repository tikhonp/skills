---
name: issue-maker
description: Turn a pasted bug, UI problem or change request (often with screenshots) into a formatted Markdown issue file, saved to a folder the user chooses (local or over ssh). Use when the user pastes a problem description and expects it written down as an issue, or says "be issue maker".
---

# Issue maker

The user pastes a rough description of a problem or change, usually with a screenshot. Your job: format it and save it as one issue file. Don't fix anything, and don't ask questions about the issue itself unless the request is truly ambiguous.

## Destination (ask first)

Never assume where issues go. Before writing the first issue of a session, ask with AskUserQuestion:

- **Where should new issues be saved?** A folder on this machine (`/path/to/folder`) or on another host (`ssh-host:/path/to/folder`, e.g. `100.64.0.19:~/projects/sandbox/proxier-notes`).
- Offer as options: the destination used earlier in this conversation (first, marked Recommended), any destination the user's memory lists for issues, and let "Other" take a custom one.

Then:

- Reuse the chosen destination for every following issue in the session. Ask again only if the user says to switch, or names a different project/folder.
- Choosing a destination authorizes writing inside that folder only. Touch nothing else on the host. For a remote, run a read-only check first (`ssh host 'ls folder'`). If the folder doesn't exist, say so and ask before creating it.
- Screenshots go in `assets/` inside the destination folder.

## Numbering and naming

- Issue file: `NNN-kebab-title.md`. Screenshot: `assets/NNN-short-name.png` (same NNN as the issue; two screenshots → two names, e.g. `NNN-...-button-spacing.png`, `NNN-...-qr-spacing.png`).
- NNN = highest number found in the destination folder **and** its `assets/`, plus 1 (zero-padded to 3 digits). Numbering is per destination.
- If the folder is empty, start at `001`.
- Fixed issues may be deleted automatically by the user's workflow, so gaps in numbering are normal. Never mention missing issues or offer to recreate them.

## Steps

1. Get the destination (see above) and look at any attached image (Read its path) so the description reflects what is actually visible.
2. List the destination and its `assets/` to get the next number.
3. Write the issue to the scratchpad directory first, then copy it into place (`cp` locally, `scp` for a remote). This avoids shell-quoting problems with apostrophes in heredocs over ssh.
4. Copy each screenshot to `assets/NNN-name.png`.
5. List the destination to confirm, then report.

## Issue format

Markdown, written in English unless the user writes the issue in another language. Concise. Drop sections that don't apply.

```
# Short imperative title

## Summary
One or two sentences: where (page/route) and what is wrong or wanted.

![alt](assets/NNN-name.png)

## Problem(s)
Bullets of what is actually visible/observed (name the labels, columns, text seen in the screenshot).
Use "### subsections" when there are several areas.

## Expected
What it should do/look like. Include the user's own request (e.g. "full width", "scroll to section on Validate").

## Notes
Related issues (reference by number, e.g. "see 002, 005"), things to check on other pages, zoom levels
(100/150/200/300%) and narrow windows for layout bugs. Possible causes only as clearly-labeled guesses.
```

Guidelines:

- Stay faithful to the user's words. Anything you inferred or added (extra sections, guessed causes, nice-to-haves) is kept brief and flagged in the reply so the user can veto it.
- Layout bugs: describe each overlap/wrap precisely (which label, what it collides with). Cross-reference similar layout issues already in the folder.
- If an issue's meaning depends on an assumption (e.g. which panel "server part" means, which issue "this issue" refers to), state the assumption in the reply and offer to fix.
- "Add this to this issue" means append to the existing or just-discussed issue file instead of creating a new one. If the previous issue was never written (e.g. the request was interrupted), create it with both parts and say so.
- If a change makes older issues obsolete, mention it in the new issue's Notes; don't edit the older ones unless asked.

## Reply

Short: file path, screenshot path, 2–4 bullets of what the issue says, then any assumptions or additions you made. Don't paste the whole file back.
