# kyae-skills

OpenCode and Codex skills for daily reports, Linear tickets, QA handoff, focused development workflows, and GStack.

## Install

Clone the repository with GStack, then install it for your agent host:

```sh
mkdir -p "$HOME/Repos/self"
git clone --recurse-submodules https://github.com/Kyaee/kyae-skills.git "$HOME/Repos/self/kyae-skills"
cd "$HOME/Repos/self/kyae-skills"
cd .agents/skills/gstack
./setup --host codex --prefix
```

Replace `codex` with `opencode`, `cursor`, `factory`, `kiro`, or `claude` as needed. Restart the agent afterward. For the full local-skills installer, every available skill, examples, updates, and troubleshooting, read [the GStack and Kyae Skills Guide](docs/gstack-guide.md).

## Commands

| Command | Use |
| --- | --- |
| `/eod` | Evidence-based end-of-day report from Codex and OpenCode sessions. |
| `/anthony <request>` | Create a Linear issue with team, requester, cycle, and acceptance criteria. |
| `/qa-ticket <issue>` | Prepare a QA child issue from a parent issue and PR evidence. |

The repository also includes skills for frontend design, documentation, teaching, code review, skill authoring, technical spikes, writing, and Todoist diary sync. The full list is in [the guide](docs/gstack-guide.md#kyae-skills-reference).

## Develop

Edit the files under `.agents/`, then restart OpenCode to load the change.
