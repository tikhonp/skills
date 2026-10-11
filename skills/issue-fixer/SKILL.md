---
name: issue-fixer
description: Work through a folder of Markdown issue files (as written by issue-maker), fixing each one in a fresh subagent, one at a time, then verifying its commit and clearing the issue. Asks how to configure itself first (issues folder, repo, commit and cleanup policy, one pass or keep polling). Use when the user says "fix the issues in <folder>", "run the issue fixer", "work through the notes folder", or invokes /issue-fixer.
---

# Issue fixer

You are the coordinator. You don't fix issues yourself: each issue gets a **fresh general-purpose subagent** (a new context, never a fork), and you check its work, clear the issue and start the next one. Issues run **one at a time**, because they all commit to the same working tree and often touch the same files.

## 1. Configure (ask first)

Never assume the setup. Before touching any issue, look for a saved config at `<issues folder>/.issue-fixer.md` if the user already named the folder. If there is one, show it in a few lines and ask with AskUserQuestion: **Use this config?** (Use it, Recommended / Change some settings / Start over).

Otherwise, or when the user wants changes, ask with AskUserQuestion. Put at most 4 questions in one call and send a second call if needed. Make the recommended option the first one. Fill the options from what you can see: the current working directory, the git remote, `ls ~/projects/sandbox`, the user's memory, and the destination issue-maker used earlier in this conversation.

1. **Issues folder.** A local path, or `host:/path` for one reached over ssh. Offer folders you found that hold `NNN-*.md` files.
2. **Repository to fix in.** Default: the current working directory if it is a git repo.
3. **Commit policy.**
   - Commit on the current branch, don't push (Recommended for a solo repo).
   - A branch per issue, don't push.
   - A branch per issue, push, and open a PR.
   - Don't commit; leave the changes for me.
4. **When an issue is done.**
   - Delete the issue file and its `assets/NNN-*` files (Recommended).
   - Move them to `done/` inside the issues folder.
   - Leave them, and record the issue as done in the config file's log.
5. **Run mode.**
   - Work through what's there now, then stop (Recommended).
   - Keep polling for new issues: start `/loop` in dynamic mode with the same request, so it wakes up and checks the folder.
6. **Ambiguous issues** (the issue asks for clarification, or doesn't name the defect).
   - Let the agent decide: it reproduces the problem and reports exactly which defects it chose to fix, and I correct afterwards (Recommended).
   - Skip the issue and ask me at the end.
7. **Extra agent instructions** (optional, through "Other"): for example "verify UI in a real browser", "no Co-Authored-By", a model or effort for the agents.

Derive everything else from the repo, and don't ask about it: its CLAUDE.md or AGENTS.md, its check, lint and build commands, its commit style (`git log -5`), and how it verifies UI.

Then write the answers to `<issues folder>/.issue-fixer.md` (a dotfile, so it never counts as an issue). Use the format in **Config file** below. Tell the user in one line that it was saved and how to change it ("say: reconfigure the issue fixer").

## 2. Find the issues

- Issues are `*.md` files in the folder itself. Ignore `assets/`, `done/` and dotfiles. Work in filename order (`001-…`, `002-…`).
- Read every issue and look at its images before you start, so you can brief each agent well. Example: a screenshot of the web UI's bottom bar, in an issue that calls it a "TUI", is worth a line of context in the brief.
- Remote folder: read it over ssh (`ssh host 'ls path'`), and copy each issue and its assets into the scratchpad (`scp`) for the agent to read. Give the agent the local copies.
- Empty folder: in one-pass mode, say so and stop. In polling mode, schedule the next check (1200 s or more) and stop for now.

## 3. Run one issue

Before you start, tell the user in one line which issue you're starting. Then spawn the agent with **Agent** (`subagent_type: general-purpose`, run in the background), using the prompt template below filled in from the config. Wait for its report. Don't poll it, and don't start the next issue while it runs.

### Agent prompt template

> You are fixing one issue in the repo at `<REPO>`. Read `<REPO>/CLAUDE.md` (or AGENTS.md) first and follow it exactly: the docs it says to read before touching code, its rules, its commands and its commit rules.
>
> The issue file is `<ISSUE PATH>`. Images it references are in `<ASSETS DIR>` (`<NNN>-*`); look at them. Read the issue in full: it is the spec. Do NOT modify or delete anything in the issues folder; the coordinator does that.
>
> `<CONTEXT: anything you learned that the issue gets wrong or leaves vague, e.g. which screen it really is, plus the ambiguity rule from the config: "the user did not name the defect; reproduce it, fix every real defect you find, and list in your report which defects you identified and why" or "if the defect is unclear, make no change and say what you'd need to know".>`
>
> Requirements:
> - Fix the issue fully, meeting every expectation in it, including its notes. Stay within the issue's scope: no unrelated refactors. Build on recent commits (`git log -8`); don't undo them. For a bug, reproduce it first, find the root cause and add a regression test.
> - Verify on the real artifact the way the repo's docs describe (run it, drive the UI in a real browser at several widths for layout work). Clean up any throwaway code, processes and containers.
> - Update tests and docs in the same commit where behaviour changes, if the repo's rules require it.
> - Run the repo's check, lint and build commands, and fix any failure.
> - `<COMMIT POLICY, in words: "Commit directly on <branch> in the style of recent commits. Do not push." / "Create branch fix/<NNN>-<slug> from <base>, commit there, do not push." / "... push it and open a PR with gh." / "Do not commit; leave the changes in the working tree.">` If the issue is already fixed, change nothing and say so with evidence.
> - `<EXTRA INSTRUCTIONS from the config>`
> - If something can't be done, fix what you can and say so plainly.
>
> When done, reply with: the commit SHA (or "no commit needed", or the branch/PR), the root cause if it's a bug, the defects you identified if the issue was vague, a short summary of what changed, the check, lint and test results, and anything left undone or unverified.

## 4. Verify, clear, continue

When the agent reports, read the report as the agent's claims, not as facts. Check them yourself:

- The commit is there: `git -C <REPO> merge-base --is-ancestor <sha> <branch>` (or the branch or PR exists, matching the policy).
- `git -C <REPO> status --short` is clean (no leftover throwaway files), unless the policy is "don't commit".
- No throwaway containers or processes the agent started are still running (`docker ps`).

If a check fails, resume that same agent with SendMessage and tell it what's wrong; don't fix it yourself. If the agent reports "no commit needed" with evidence, count the issue as done.

Then clear the issue the way the config says: delete it, move it to `done/`, or append a line to the config's log. Over ssh, run the same command on the host. If a permission check refuses the delete, don't work around it: tell the user which files are left, then carry on.

Append to the config file's **Log**: `- <NNN> <title> — <sha or "no commit"> (<date>)`.

Tell the user in a few lines what landed, and in particular what the agent decided on its own (its reading of a vague issue) and what it left undone. Then start the next issue.

## 5. Finish

When the folder is empty (in one-pass mode), report briefly:

- Each issue with its commit and a one-line summary.
- Everything left undone or unverified, across all agents.
- Files you couldn't clear.

Don't push or deploy unless the commit policy says so.

## Config file

`<issues folder>/.issue-fixer.md`:

```markdown
# Issue fixer config

- Issues folder: /Users/me/projects/sandbox/app-notes
- Repo: /Users/me/projects/app
- Commit: on main, no push
- Done issues: delete
- Run mode: one pass
- Ambiguous issues: agent decides and reports
- Extra agent instructions: verify UI in the chromedp container; no Co-Authored-By

## Log

- 001 Fix g key suggestions — 29b9b20 (2026-10-11)
```

The config is in plain words on purpose: the user can edit it by hand, and you read it the same way.
