# kyae-skills

OpenCode skills and slash commands for daily reports, Linear tickets, QA handoff, and focused development workflows.

## Install

Copy and run this in a terminal. It installs every skill and command, then OpenCode discovers them after a restart.

```sh
mkdir -p "$HOME/Repos/self"
git clone https://github.com/Kyaee/kyae-skills.git "$HOME/Repos/self/kyae-skills"
cd "$HOME/Repos/self/kyae-skills"

mkdir -p "$HOME/.agents/skills" "$HOME/.config/opencode/commands"
repo_dir="$PWD"

for skill_file in .agents/skills/*/SKILL.md; do
  [ -f "$skill_file" ] || continue
  skill_dir="${skill_file%/SKILL.md}"
  ln -sfn "$repo_dir/$skill_dir" "$HOME/.agents/skills/$(basename "$skill_dir")"
done

for command_file in .agents/commands/*.md; do
  [ -f "$command_file" ] || continue
  ln -sfn "$repo_dir/$command_file" "$HOME/.config/opencode/commands/$(basename "$command_file")"
done
```

Restart OpenCode after installation. Later updates only need `git pull` and an OpenCode restart.

## Commands

| Command | Use |
| --- | --- |
| `/eod` | Evidence-based end-of-day report from Codex and OpenCode sessions. |
| `/anthony <request>` | Create a Linear issue with team, requester, cycle, and acceptance criteria. |
| `/qa-ticket <issue>` | Prepare a QA child issue from a parent issue and PR evidence. |

The repository also includes skills for frontend design, documentation, teaching, code review, and skill authoring. Read each `SKILL.md` for its scope.

## Develop

Edit the files under `.agents/`, then restart OpenCode to load the change.
