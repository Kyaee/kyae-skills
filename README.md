List of skills from skill.sh

- executive-document-secretary
- eod
- find-skills
- frontend-design
- grill-me
- skill-creator
- teach
- web-design-guidelines
- ai-no-slop

Canonical paths:

- Skill: `.agents/skills/eod/SKILL.md`
- Command: `.agents/commands/eod.md`

Reproducible global installation:

```sh
mkdir -p "$HOME/.agents/skills" "$HOME/.config/opencode/commands"
ln -sfn "/home/kyae-dev/Repos/self/kyae-skills/.agents/skills/eod" "$HOME/.agents/skills/eod"
ln -sfn "/home/kyae-dev/Repos/self/kyae-skills/.agents/commands/eod.md" "$HOME/.config/opencode/commands/eod.md"
```

Restart OpenCode after installation, or after source changes that need rediscovery.

Examples:

```text
/eod
/eod 2026-08-15
/eod 2026-08-15 timezone=Asia/Singapore write=/tmp/eod-2026-08-15.md
```
