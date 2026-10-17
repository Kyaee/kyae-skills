# GStack and Kyae Skills Guide

This repository contains two skill packs:

- **GStack**, vendored as `.agents/skills/gstack`. It supplies engineering, design, QA, release, browser, security, and iOS workflows.
- **Kyae Skills**, the first-level directories in `.agents/skills/`. They add local workflows for Linear, end-of-day reporting, formal documents, teaching, frontend work, technical spikes, writing, and Todoist diary sync.

GStack is installed for a specific coding-agent host. Kyae Skills are linked from this checkout so they can be kept under version control and updated with `git pull`.

## What you need

- Git
- [Bun](https://bun.sh/) 1.0 or later for GStack setup
- At least one supported coding agent, such as Codex CLI or OpenCode

On Linux and Windows, browser-related skills use GStack's bundled Chromium fallback. On macOS, they use Aside first when it is installed and running.

## Install from this repository

Clone with submodules. GStack is a submodule, so a normal clone leaves its files unavailable.

```sh
mkdir -p "$HOME/Repos/self"
git clone --recurse-submodules https://github.com/Kyaee/kyae-skills.git "$HOME/Repos/self/kyae-skills"
cd "$HOME/Repos/self/kyae-skills"
```

If you already cloned the repository, initialize it instead:

```sh
cd "$HOME/Repos/self/kyae-skills"
git submodule update --init --recursive
```

### Install GStack for Codex CLI

```sh
cd "$HOME/Repos/self/kyae-skills/.agents/skills/gstack"
./setup --host codex --prefix
```

This generates and installs skills under `${CODEX_HOME:-$HOME/.codex}/skills/`. With `--prefix`, use names such as `/gstack-review`; it avoids collisions with other packs. Omit `--prefix` to use short names such as `/review`.

### Install GStack for OpenCode

```sh
cd "$HOME/Repos/self/kyae-skills/.agents/skills/gstack"
./setup --host opencode --prefix
```

This installs generated skills under `~/.config/opencode/skills/`. Restart OpenCode after setup.

### Other supported hosts

Replace the host name in the command above with one of:

| Host | Flag | Install location |
| --- | --- | --- |
| Codex CLI | `codex` | `${CODEX_HOME:-~/.codex}/skills/` |
| OpenCode | `opencode` | `~/.config/opencode/skills/` |
| Cursor | `cursor` | `~/.cursor/skills/` |
| Factory Droid | `factory` | `~/.factory/skills/` |
| Kiro | `kiro` | `~/.kiro/skills/` |
| Claude Code | `claude` | `~/.claude/skills/` |

For automatic host detection, run `./setup` without `--host`. See `./setup --help` for the current options.

### Install this repository's local skills and commands

These links let the local pack update when you pull this repository.

```sh
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

Restart your agent after adding links. The local commands are `/anthony`, `/eod`, and `/qa-ticket`.

## How to use a skill

Start a new agent turn with the skill command, then give it the concrete task and any constraints. For example:

```text
/gstack-investigate The checkout button returns 500 for signed-in users. Reproduce it first; do not change code until you identify the cause.
```

```text
/gstack-review Review the current diff for production bugs. Report findings only; do not edit files.
```

```text
/eod Make my EOD for 2026-09-14 in Asia/Singapore from this repository's work.
```

Use the prefixed name if setup used `--prefix`; otherwise remove `gstack-` from the examples. A skill is guidance, not a bypass for permissions: commands that modify a remote, deploy, send messages, or delete data still require the appropriate authority.

## Choose the right workflow

| Goal | Start with | Follow with |
| --- | --- | --- |
| Shape a new product or feature | `/gstack-office-hours` | `/gstack-autoplan`, then implementation |
| Turn a vague request into a build-ready issue | `/gstack-spec` | implementation, then `/gstack-ship` |
| Find the cause of a defect | `/gstack-investigate` | fix and focused verification |
| Review a change before merging | `/gstack-review` | `/gstack-ship` |
| Test a live app without changing it | `/gstack-qa-only` | file the reported issues |
| Test and fix a live app | `/gstack-qa` | `/gstack-ship` |
| Deploy an approved pull request | `/gstack-land-and-deploy` | `/gstack-canary` |
| Protect a sensitive task | `/gstack-guard path/to/area` | `/gstack-unfreeze` when finished |

## GStack skill reference

The names below assume `--prefix`. If you chose `--no-prefix`, remove `gstack-`.

### Plan and design

| Skill | Use it for |
| --- | --- |
| `/gstack-office-hours` | Challenge and clarify a product idea before planning. |
| `/gstack` | Route a broad GStack request to the most suitable workflow. |
| `/gstack-autoplan` | Run CEO, design, developer-experience, and engineering plan reviews in order. |
| `/gstack-spec` | Turn a vague request into an executable specification and issue. |
| `/gstack-plan-ceo-review` | Test whether the plan solves the right customer problem. |
| `/gstack-plan-eng-review` | Lock down architecture, data flow, edge cases, and tests. |
| `/gstack-plan-design-review` | Evaluate a proposed UI or product design before implementation. |
| `/gstack-plan-devex-review` | Review the onboarding and usability of a developer-facing plan. |
| `/gstack-plan-tune` | Adjust how often GStack asks for decisions. |
| `/gstack-design-consultation` | Create a product design direction and design system. |
| `/gstack-design-shotgun` | Generate and compare multiple visual directions. |
| `/gstack-design-html` | Turn an approved design direction into production HTML and CSS. |

### Build, investigate, and review

| Skill | Use it for |
| --- | --- |
| `/gstack-investigate` | Reproduce and trace a bug before proposing a fix. |
| `/gstack-review` | Find production-risk bugs in the current change before landing it. |
| `/gstack-claude` | Ask Claude Code for an independent review, challenge, or consultation. |
| `/gstack-qa` | Exercise a real web app, fix confirmed bugs, and verify the result. |
| `/gstack-qa-only` | Perform the same QA process but report findings only. |
| `/gstack-design-review` | Audit a live interface and fix visual or interaction problems. |
| `/gstack-devex-review` | Test a real developer onboarding flow and identify friction. |
| `/gstack-health` | Run a code-health view across types, lint, tests, and dead code. |
| `/gstack-benchmark` | Baseline and compare web performance regressions. |
| `/gstack-benchmark-models` | Compare skill output across Claude, GPT, and Gemini. |
| `/gstack-cso` | Run a security audit using OWASP Top 10 and STRIDE. |

### Browser and research

| Skill | Use it for |
| --- | --- |
| `/gstack-browse` | Open, inspect, click through, screenshot, and check a website. |
| `/gstack-open-gstack-browser` | Open the visible GStack browser. |
| `/gstack-scrape` | Read structured webpage data using the available browser session. |
| `/gstack-skillify` | Save a successful scrape flow as a reusable browser skill. |
| `/gstack-setup-browser-cookies` | Import browser cookies for authenticated fallback-browser testing. |
| `/gstack-pair-agent` | Give another AI agent scoped access to the GStack browser. |

### Ship and operate

| Skill | Use it for |
| --- | --- |
| `/gstack-ship` | Run checks, prepare a commit and pull request, and push with review evidence. |
| `/gstack-land-and-deploy` | Merge an approved pull request, wait for deployment, and check production. |
| `/gstack-canary` | Monitor a deployed application for failures and regressions. |
| `/gstack-landing-report` | See the read-only status of the workspace-aware ship queue. |
| `/gstack-setup-deploy` | Detect and configure deployment information for the release workflow. |
| `/gstack-upgrade` | Update the GStack installation. |
| `/gstack-context-save` | Save git state, decisions, and remaining work for a later session. |
| `/gstack-context-restore` | Resume a saved GStack context. |
| `/gstack-learn` | Inspect or manage learned project patterns and preferences. |
| `/gstack-retro` | Produce a weekly engineering retrospective. |
| `/gstack-setup-gbrain` | Set up GBrain for cross-session or cross-machine memory. |
| `/gstack-sync-gbrain` | Refresh GBrain's knowledge of the current repository. |

### Documentation, safety, and utilities

| Skill | Use it for |
| --- | --- |
| `/gstack-document-generate` | Write missing project documentation from code. |
| `/gstack-document-release` | Bring documentation in line with a shipped change. |
| `/gstack-make-pdf` | Render Markdown as a publication-quality PDF, HTML, or DOCX. |
| `/gstack-diagram` | Create Mermaid, Excalidraw, SVG, and PNG diagrams from a description. |
| `/gstack-careful` | Add warnings and hard stops for dangerous commands. |
| `/gstack-freeze` | Restrict edits to one directory for the session. |
| `/gstack-guard` | Enable both destructive-command protection and an edit boundary. |
| `/gstack-unfreeze` | Remove the active edit boundary. |

### iOS workflows

| Skill | Use it for |
| --- | --- |
| `/gstack-ios-qa` | Test a real iPhone-connected SwiftUI application. |
| `/gstack-ios-fix` | Investigate and fix an iOS bug with regression snapshots. |
| `/gstack-ios-design-review` | Audit an iOS app against Apple Human Interface Guidelines. |
| `/gstack-ios-clean` | Remove DebugBridge wiring before a release build. |
| `/gstack-ios-sync` | Regenerate the iOS debug bridge from current templates. |

## Kyae Skills reference

| Skill or command | Use it for |
| --- | --- |
| `/anthony` | Create a well-scoped Linear issue with team, requester, cycle, and acceptance criteria. |
| `/eod` | Produce an evidence-based end-of-day report from Codex and OpenCode records. |
| `/qa-ticket` | Prepare a QA child issue from a Linear parent issue and pull-request evidence. |
| `create-technical-spike` | Create a time-boxed research document for a technical decision, with findings and follow-up actions. |
| `executive-document-secretary` | Draft or revise formal workplace, administrative, or institutional documents. |
| `find-skills` | Find or install a skill for a capability you need. |
| `frontend-design` | Design a distinctive, intentional interface rather than a default-looking UI. |
| `grill-me` | Pressure-test a plan or design through a direct interview. |
| `grill-with-docs` | Pressure-test a plan or design while documenting decisions and domain terms. |
| `no-ai-slop` | Edit prose using concrete claims, plain wording, and examples of writing to avoid. |
| `rossmann-voice` | Write prose using Louis Rossmann's style, with specific evidence and varied sentence lengths. |
| `skill-creator` | Create, revise, evaluate, or optimize a skill. |
| `teach` | Learn a concept or workflow within the active workspace. |
| `todoist-diary-sync` | Update Todoist tasks and mirror the resulting task data into today's diary. |
| `web-design-guidelines` | Audit UI code for web-interface and accessibility best practices. |

`grill-with-docs` calls the separate `/grilling` and `/domain-modeling` skills. `todoist-diary-sync` needs Todoist tools and the existing `/read-diary` workflow. Those dependencies must be available in the host workspace; they are not bundled with these skills.

## Update and troubleshoot

Update the repository and its submodule, then rerun setup for your host:

```sh
cd "$HOME/Repos/self/kyae-skills"
git pull --recurse-submodules
git submodule update --init --recursive
cd .agents/skills/gstack
./setup --host codex --prefix
```

Replace `codex` with your host, then restart the agent. If a GStack skill does not appear, check the generated install directory for your host and rerun `./setup --host <host>`. If a browser skill reports that Aside is unavailable, that is expected on Linux and Windows: it uses GStack's fallback Chromium after setup completes.

## Source of truth

GStack's own skill descriptions and current setup options are in `.agents/skills/gstack/AGENTS.md` and `./setup --help`. This guide documents the vendored revision in this repository; refresh it when the GStack submodule is updated.
