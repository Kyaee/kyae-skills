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

## Skills in this repository

This repository contains the Kyae skill pack, two additional nested skills, three slash commands, and the vendored GStack suite. The table below describes the Kyae skills by the job each one performs. The [GStack catalog](docs/gstack-guide.md#gstack-skill-reference) defines each GStack skill separately.

| Skill | What it does | Use it when |
| --- | --- | --- |
| [`anthony`](.agents/skills/anthony/SKILL.md) | Turns an issue request into a Linear issue with the right team, requester, cycle, urgency, and acceptance criteria. | You want a scoped Linear issue created. |
| [`create-technical-spike`](.agents/skills/create-technical-spike/SKILL.md) | Produces a time-boxed research document to resolve a development decision before implementation. | A technical choice needs investigation, findings, and follow-up actions. |
| [`eod`](.agents/skills/eod/SKILL.md) | Builds an evidence-based end-of-day report from Codex and OpenCode work records. | You need a daily summary for a date, timezone, or project. |
| [`executive-document-secretary`](.agents/skills/executive-document-secretary/SKILL.md) | Drafts, revises, proofreads, and formats formal administrative and workplace documents, including memos, letters, proposals, policies, MOAs, and MOUs. | You need a professional institutional or business document prepared or improved. |
| [`find-skills`](.agents/skills/find-skills/SKILL.md) | Finds existing agent skills or helps identify and install one for a requested capability. | You ask whether a skill exists for a task or want to extend an agent. |
| [`frontend-design`](.agents/skills/frontend-design/SKILL.md) | Guides visual direction, typography, and interface choices for distinctive, intentional UI. | You are building a new interface or substantially reshaping one. |
| [`grill-me`](.agents/skills/grill-me/SKILL.md) | Runs a focused, persistent interview to expose assumptions and sharpen a plan or design. | You want your proposal pressure-tested through questions. |
| [`grill-with-docs`](.agents/skills/grill-with-docs/SKILL.md) | Pressure-tests a plan or design while recording decisions in architecture decision records and a glossary. | You want both a sharper proposal and written design rationale. It depends on the separate `grilling` and `domain-modeling` skills. |
| [`no-ai-slop`](.agents/skills/no-ai-slop/SKILL.md) | Applies concrete, plain-language writing rules and flags generic or unsupported prose. | You are writing or editing prose and want it to sound specific and grounded. |
| [`qa-ticket`](.agents/skills/qa-ticket/SKILL.md) | Prepares a QA child issue from a Linear parent and linked pull-request evidence, with changes and unchecked QA criteria. | You need a reviewable QA handoff in Linear. |
| [`rossmann-voice`](.agents/skills/rossmann-voice/SKILL.md) | Shapes general prose in Louis Rossmann's voice, using claim-then-proof structure, concrete evidence, and varied sentence lengths. | You explicitly want prose in that voice. |
| [`skill-creator`](.agents/skills/skill-creator/SKILL.md) | Creates and revises skills, then evaluates triggering and task performance. | You are building a skill or improving one with evals. |
| [`teach`](.agents/skills/teach/SKILL.md) | Teaches a concept or workflow in the context of the active workspace. | You want a guided explanation grounded in the current project. |
| [`todoist-diary-sync`](.agents/skills/todoist-diary-sync/SKILL.md) | Updates Todoist tasks and mirrors the resulting list into the current day's diary inputs. | You ask to change Todoist tasks and keep the diary in sync. It needs Todoist tools and the separate `/read-diary` workflow. |
| [`web-design-guidelines`](.agents/skills/web-design-guidelines/SKILL.md) | Reviews UI code for web interface and accessibility guideline compliance. | You want a code-level UI, UX, or accessibility audit. |
| [`improve-codebase-architecture`](.agents/skills/.agents/skills/improve-codebase-architecture/SKILL.md) | Scans a codebase for architectural deepening opportunities, presents a visual HTML report, and interviews you about a selected opportunity. | You want to identify high-value structural improvements before changing code. |
| [`vercel-react-best-practices`](.agents/skills/.agents/skills/vercel-react-best-practices/SKILL.md) | Provides Vercel Engineering guidance for React and Next.js performance, including components, data fetching, and bundle size. | You are writing, reviewing, or refactoring React or Next.js code. |

### Slash commands

These commands are separate from skills and are installed for OpenCode by the local-skills setup:

| Command | What it does |
| --- | --- |
| [`/anthony <request>`](.agents/commands/anthony.md) | Creates a Linear issue using the request and resolved team metadata. |
| [`/eod`](.agents/commands/eod.md) | Generates an end-of-day report from work records. |
| [`/qa-ticket <issue>`](.agents/commands/qa-ticket.md) | Prepares the QA child issue and criteria from parent issue and PR evidence. |

GStack is vendored as a submodule and installed separately for each agent host. Its [skill reference](docs/gstack-guide.md#gstack-skill-reference) describes the individual planning, coding, review, QA, browser, release, security, documentation, and iOS workflows. The guide also covers [installing local skills and commands](docs/gstack-guide.md#install-this-repositorys-local-skills-and-commands), dependencies, updates, and troubleshooting.

## Develop

Edit the files under `.agents/`, then restart OpenCode to load the change.
