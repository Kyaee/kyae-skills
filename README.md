List of skills from skill.sh

- executive-document-secretary
- eod
- find-skills
- frontend-design
- grill-me
- qa-ticket
- skill-creator
- teach
- web-design-guidelines
- ai-no-slop

Canonical paths:

- Skill: `.agents/skills/eod/SKILL.md`
- Command: `.agents/commands/eod.md`
- Skill: `.agents/skills/qa-ticket/SKILL.md`
- Tests: `.agents/skills/qa-ticket/TESTS.md`
- Command: `.agents/commands/qa-ticket.md`

Reproducible global installation:

```sh
mkdir -p "$HOME/.agents/skills" "$HOME/.config/opencode/commands"
Make sure to rename the brackets based on your own folder
```bash
ln -sfn "{your-folder}/.agents/skills/eod" "$HOME/.agents/skills/eod"
ln -sfn "{your-folder}/Repos/self/kyae-skills/.agents/commands/eod.md" "$HOME/.config/opencode/commands/eod.md"
ln -sfn "{your-folder}/Repos/self/kyae-skills/.agents/skills/qa-ticket" "$HOME/.agents/skills/qa-ticket"
ln -sfn "{your-folder}/kyae-dev/Repos/self/kyae-skills/.agents/commands/qa-ticket.md" "$HOME/.config/opencode/commands/qa-ticket.md"
```

Restart OpenCode after installation, or after source changes that need rediscovery.

`/qa-ticket` targets the `qa-ticket` skill for the `Quality Assurance` team. Preview is the default. Explicit `create` is the Linear side effect.

Examples:

```text
/eod
/eod 2026-08-15
/eod 2026-08-15 timezone=Asia/Singapore write=/tmp/eod-2026-08-15.md
/qa-ticket QO-60
/qa-ticket QO-60 pr=https://github.com/Quanby-OJT/PMDv2/pull/5
/qa-ticket QO-60 create
```
