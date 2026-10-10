# skills

My own collection of [Claude Code](https://claude.com/claude-code) skills.

## What's inside

- **[issue-maker](skills/issue-maker/SKILL.md)** — paste a bug or change request (screenshots welcome) and it writes a clean Markdown issue file to a folder you pick, local or over ssh.

## Install

In Claude Code:

```
/plugin marketplace add TikhonP/skills
/plugin install tikhon@tikhon-skills
```

Skills then show up as `/tikhon:issue-maker`. To get new skills later, update the plugin from `/plugin`.

## Adding a skill

Make a folder `skills/<name>/` with a `SKILL.md` inside, push, done.

## License

[AGPL-3.0](LICENSE.md)
