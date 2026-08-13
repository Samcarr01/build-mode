# Claude Code setup - verified syntax

Checked against the live docs at code.claude.com on 7 August 2026. Claude Code moves fast, so if something below does not behave as described, check the docs rather than assuming the user has done something wrong. The three pages that matter: `/docs/en/memory`, `/docs/en/skills`, `/docs/en/permission-modes`.

Wrong syntax here fails silently. The user will not notice a skill that never loads. Copy the shapes exactly.

**Generating the doc pack?** You need sections 1 to 4. Sections 5 to 8 are advice for the user
about how to run Claude Code day to day - read them when they ask, not while writing files.

## Contents

1. [CLAUDE.md](#claudemd)
2. [Project rules](#project-rules)
3. [Project skills - the slash commands](#project-skills)
4. [settings.json](#settingsjson)
5. [Permission modes and plan mode](#permission-modes)
6. [Context hygiene](#context-hygiene)
7. [Optional power-ups](#optional-power-ups)
8. [What is loaded when](#what-is-loaded-when)

---

## CLAUDE.md

Loaded into context at the start of every session. That is the whole point and also the whole cost.

**Locations, loaded broadest first, so the most specific wins:**

| Scope | Path |
|---|---|
| User, all projects | `~/.claude/CLAUDE.md` |
| Project, committed | `./CLAUDE.md` or `./.claude/CLAUDE.md` |
| Project, personal, gitignored | `./CLAUDE.local.md` |

Subdirectory `CLAUDE.md` files load on demand when Claude Code reads a file in that directory.

**Size: under 200 lines.** This is not a style preference. Longer files measurably reduce how well Claude Code follows them, and they burn context every single session. When it grows past 200, move things into `.claude/rules/` with `paths:` scoping, or into a skill.

**Imports:** `@path/to/file` anywhere in the file pulls that file in. Relative paths resolve from the file containing the import. Max four hops deep. Backtick a path to mention it without importing: `` `@README` ``.

Important: **imports load at launch too.** They organise, they do not save context. So do not `@import` `docs/ROADMAP.md` into `CLAUDE.md` - that would drag the whole roadmap into every session. Point at it instead:

```markdown
Before starting a task, read `docs/ROADMAP.md` and `docs/PROGRESS.md`.
```

Claude Code reads it when it needs it. That is the difference between a lean session and one that compacts halfway through the first task.

**HTML comments** (`<!-- like this -->`) are stripped before the file enters context. Useful for notes to the user that Claude Code should not spend tokens on.

**Related commands:** `/init` generates a starter CLAUDE.md from the codebase, or proposes improvements if one exists. `/memory` opens memory files for editing. `/context` shows what actually loaded - the honest check when an instruction is being ignored.

**Auto memory** is separate and on by default: Claude Code writes its own notes to `~/.claude/projects/<project>/memory/MEMORY.md`, first 200 lines loaded each session. It is machine-local and outside the repo, so Cowork cannot read it. That is exactly why the project keeps its own `docs/LEARNINGS.md` - both tools need to see it.

---

## Project rules

`.claude/rules/*.md`. Each file one topic. Without frontmatter they load every session, same as `CLAUDE.md`. With a `paths:` list they load only when Claude Code touches a matching file, which is how you get design rules that cost nothing on a backend task:

```markdown
---
paths:
  - "src/**/*.{tsx,jsx}"
  - "app/**/*.tsx"
---

# UI rules

Read `docs/DESIGN.md` before building any screen.
Use the tokens defined there. Do not introduce new colours or font sizes.
No `shadow-lg`, no purple-to-blue gradients, no emoji in headings.
```

Glob patterns support `*`, `**` and brace expansion `{a,b}`. Rules with `paths:` do not survive `/compact` - they reload next time a matching file is read.

---

## Project skills

Custom commands have been merged into skills. `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both give you `/deploy`; skills are the recommended form because they can carry supporting files. **Use `.claude/skills/<name>/SKILL.md`.**

**The command name comes from the directory name.** In a project or personal skill the `name` field is only a display label. So `/next` needs to live at `.claude/skills/next/SKILL.md` - naming the directory something else and setting `name: next` will not work.

**Frontmatter that matters here:**

| Field | Use |
|---|---|
| `description` | What it does and when. The only recommended field. |
| `disable-model-invocation: true` | Only the user can run it. Use on anything with side effects, so Claude Code does not decide to run `/checkpoint` mid-task. |
| `user-invocable: false` | Only Claude Code can load it. For background knowledge, not actions. |
| `allowed-tools` | Pre-approves tools for the turn that invokes it, so Claude Code does not stop to ask. Space or comma separated: `Bash(git add *) Bash(git commit *)`. It **grants**, it does not restrict - every other tool stays callable - so a short list here never limits what a skill can do. Clears on the user's next message. |
| `argument-hint` | Autocomplete hint, e.g. `[task-id]`. |
| `model`, `effort` | Override for the turn. `effort` takes `low`, `medium`, `high`, `xhigh`, `max`. |
| `context: fork` | Runs in a subagent so it does not pollute the main context. |

**Arguments.** `$ARGUMENTS` is everything passed. Indexed access is **zero-based**: `$0` is the first argument, `$1` the second. `$ARGUMENTS[0]` is the long form of `$0`. This trips people up who expect shell-style `$1` first - it is not shell.

**Dynamic context.** `` !`command` `` runs the shell command before the skill content reaches Claude Code, and the output replaces the placeholder. This is how `/next` can show live git state.

**Useful variables:** `${CLAUDE_PROJECT_DIR}` (project root), `${CLAUDE_SKILL_DIR}` (the skill's own folder), `${CLAUDE_SESSION_ID}`.

### The three skills to generate

**Which of these Claude Code may run itself matters, and it is easy to get backwards.**
`/next` starts work, so only the user invokes it - otherwise Claude Code decides on its own to
begin the next task. `/checkpoint` and `/blocked` are the opposite: every prompt you write
instructs Claude Code to run them, so they must stay model-invocable. Putting
`disable-model-invocation` on those two silently breaks the tracking loop, which is the
mechanism the whole system rests on.

**`.claude/skills/next/SKILL.md`**

```markdown
---
description: Do the task written in docs/next-prompt.md. Run this when starting a new task from the plan.
disable-model-invocation: true
---

## Current state

Branch and recent commits:
!`git log --oneline -5 2>/dev/null || echo "no repo yet - this is the first task"`

## Your task

Read `docs/next-prompt.md` and do what it says.

Before you start, also read `CLAUDE.md`, `docs/ARCHITECTURE.md` and `docs/LEARNINGS.md`
so you are not rediscovering decisions that have already been made.

Work in plan mode by default. Show the plan and wait for approval before editing
anything. If the user cannot review a diff, the plan is the only review this gets - skip it
only for a one-file change that is obviously safe.

If any step of the task looks wrong to you, or conflicts with something in the docs,
say so before you build rather than after.

When you have finished, run `/checkpoint`.

Do not start the next task. Stop and say what you did.
```

**`.claude/skills/checkpoint/SKILL.md`**

```markdown
---
description: Write the current state of work into the tracking docs. Run at the end of every task, before /clear.
---

Update the three tracking documents to reflect what you just did. Be accurate rather
than generous - these files are the only memory the project has between sessions, and
an optimistic entry costs more than an honest one.

1. `docs/PROGRESS.md` - add an entry at the top, matching the format of the entries
   already there. Include the task ID, what changed, the files touched, and anything
   you did not finish or were unsure about.
2. `docs/ROADMAP.md` - update the status of the task you worked on. Only mark it done
   if every line of its Definition of Done is genuinely true.
3. `docs/LEARNINGS.md` - add anything that would save time next session: a gotcha, a
   version constraint, a command that works, a dead end worth not repeating. Only add
   it if it would change a decision later. Skip it otherwise.

Then give a three-line summary of where things stand.
```

**`.claude/skills/blocked/SKILL.md`**

```markdown
---
description: Write up a blocker so it can be handed back to planning. Run when stuck after two real attempts.
argument-hint: [what is blocking you]
---

You are blocked on: $ARGUMENTS

Stop trying to fix it. Write a blocker report instead - a clear description of the
problem is worth more than a third guess at the solution.

Add to the top of `docs/PROGRESS.md`:

## BLOCKED - [task ID] - [today's date]
**Symptom:** what actually happens, including the exact error text
**Tried:** each attempt and what it did
**Suspect:** your best guess at the cause and why
**Need:** the decision or information that would unblock this

Then print the same report in the chat so it can be pasted into planning.
```

---

## settings.json

`.claude/settings.json` for project-wide, `.claude/settings.local.json` for personal and gitignored. Precedence: managed, then CLI flags, then local, then project, then user.

Permission rules **merge** across scopes rather than overriding, and `deny` beats `allow`.

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run dev)",
      "Bash(npm run build)",
      "Bash(npm run lint)",
      "Bash(npm test *)",
      "Bash(git status *)",
      "Bash(git diff *)",
      "Bash(git log *)"
    ],
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Bash(rm -rf *)"
    ]
  }
}
```

Two things to know before adding more:

- **Do not set `defaultMode` in project settings.** `auto` is ignored there by design, so a repo cannot grant itself auto mode. If the user wants a default mode it goes in `~/.claude/settings.json`.
- **`.claude/` is a protected path.** Writes to it are never auto-approved except in `bypassPermissions`, and `allow` rules do not change that. The user will get a prompt when Claude Code edits its own config. That is intentional; do not try to engineer around it.

---

## Permission modes

Six modes. The config value is on the left, the label the user sees on the right.

| Value | Label | Runs without asking |
|---|---|---|
| `default` | Manual | Reads only |
| `acceptEdits` | Edit automatically | Reads, file edits, `mkdir`/`mv`/`cp`/`rm` etc. in the working directory |
| `plan` | Plan | Reads, plus classifier-approved commands when auto mode is on |
| `auto` | Auto | Everything, with a safety classifier checking each action |
| `dontAsk` | - | Only pre-approved tools. For CI. |
| `bypassPermissions` | Bypass permissions | Everything, no checks. Containers only. |

`Shift+Tab` cycles `default` -> `acceptEdits` -> `plan`. Auto and bypass slot in after plan when enabled.

**Plan mode is the single most useful habit for the user.** Claude Code researches and proposes, and does not touch source until they approve. For someone who cannot review a diff, the plan is the review. Recommend it for any task touching more than a couple of files.

Enter it with `Shift+Tab`, or prefix one prompt with `/plan`, or start with `claude --permission-mode plan`. `Ctrl+G` opens the proposed plan in an editor. On approval they pick "use auto mode" or "manually approve edits".

**On auto mode:** it makes long tasks much less annoying, and the classifier blocks the genuinely dangerous things - production deploys, migrations, force push, secrets leaving the repo. It is a reasonable default for the user once a project is past the risky early stage. It is not a substitute for reading the plan. Note the classifier also treats boundaries stated in conversation as blocks: "don't deploy until I say" actually holds.

---

## Context hygiene

The most common reason a Claude Code session goes bad is a context window full of a previous task's dead ends.

- `/clear` between unrelated tasks. Cheap, and the single highest-value habit. The doc pack means nothing is lost.
- `/compact <instruction>` to summarise mid-task, e.g. `/compact keep the API changes, drop the debugging`.
- `/context` to see what is loaded.
- `/rewind`, or `Esc` twice, to roll back code, conversation, or both. 100 checkpoints kept per session.
- Auto-compaction happens on its own when the window fills. Project-root `CLAUDE.md` is re-read from disk afterwards; nested rules and `paths:`-scoped rules are not, and reload on next match.

Build the habit into the workflow: one task, then `/checkpoint`, then `/clear`, then `/next`.

---

## Optional power-ups

Suggest these when the user is comfortable, not at kickoff. Each one is a thing that can break confusingly.

**Subagents** at `.claude/agents/<name>.md` with `name`, `description`, `tools`, `model` frontmatter. Worth it for research-heavy work: the subagent burns its own context and returns a summary. They run in the background by default and nest up to three deep.

**Hooks** in `settings.json` under a `hooks` key - shell commands at lifecycle events (`PreToolUse`, `PostToolUse`, `SessionStart`, `UserPromptSubmit`, `Stop`, `InstructionsLoaded`). Unlike CLAUDE.md instructions, hooks always run. Good for auto-formatting on edit or a `SessionStart` that prints the current task. Check the exact JSON shape at `/docs/en/hooks` before writing one; it is fiddly and a malformed hook fails quietly.

**`.mcp.json`** in the project root for project-scoped MCP servers, so Supabase and Vercel are available to Claude Code the way they are here.

**`/doctor`** runs a setup checkup: unused skills, colliding MCP servers, and a proposed trim for a bloated CLAUDE.md. Worth running when a project has been going a while.

---

## What is loaded when

Worth holding in your head when you decide where to put something.

| Thing | Loaded |
|---|---|
| `CLAUDE.md` and its `@imports` | Every session, in full |
| `.claude/rules/*.md` without `paths:` | Every session |
| `.claude/rules/*.md` with `paths:` | When a matching file is read |
| Skill `description` | Every session |
| Skill body | When invoked, then stays for the session |
| `docs/*.md` | Only when Claude Code reads them |

So: standing rules go in `CLAUDE.md`, UI rules go in a `paths:`-scoped rule, procedures go in a skill, and reference material stays in `docs/` behind a pointer.
