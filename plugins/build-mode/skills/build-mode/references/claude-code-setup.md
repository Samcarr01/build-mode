# Claude Code setup - verified syntax

Checked against the live docs at code.claude.com on 7 August 2026. Claude Code moves fast, so if something below does not behave as described, check the docs rather than assuming the user has done something wrong. The three pages that matter: `/docs/en/memory`, `/docs/en/skills`, `/docs/en/permission-modes`.

Wrong syntax here fails silently. They will not notice a skill that never loads. Copy the shapes exactly.

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

**The brainstorming step is already done.** This task was scoped and designed before it
reached you, and the answers are in those documents. Do not open a design conversation
and do not ask the user to approve a design - read the docs instead. If something here
conflicts with them, say so before you build.

Work in plan mode by default. Show the plan and wait for approval before editing
anything. The user cannot review a diff, so the plan is the only review this gets - skip it
only for a one-file change that is obviously safe.

If any step of the task looks wrong to you, or conflicts with something in the docs,
say so before you build rather than after.

When you have finished, run `/checkpoint`.

Do not start the next task. Stop and say what you did.
```

**`.claude/skills/checkpoint/SKILL.md`**

The load-bearing file. It is the only place anything is actually **executed** before a
task gets ticked - Cowork has no route to localhost on the user's machine, so if the
checks do not happen here they do not happen at all, and "done" means nothing more than
Claude Code feeling finished. Generate it with the checks in, every time. Adjust the
commands to the project's actual stack: the shapes below assume Node and npm, and a
Python or Swift project needs its own equivalents.

```markdown
---
description: Prove the task is done, then record it. Run at the end of every task, before /clear.
allowed-tools: Bash(npm run build) Bash(npm run lint) Bash(npx tsc --noEmit) Bash(git add *) Bash(git commit *) Bash(git push *)
# Generated by build-mode. Keep the checks in section 1 - they are the point of this file.
---

Two halves. Do not skip to the second one - evidence first, paperwork second.

## 1. Prove it

Run these before you write a single word into the docs.

- `npm run build` - it must complete.
- `npx tsc --noEmit` - no type errors. Skip only if the project is not TypeScript.
- If the project has tests, run them.
- Then check the thing you actually built. Start the dev server and walk the flow this
  task added with whatever browser tool this project has - the Playwright MCP, or the
  `webapp-testing` skill. Click it the way a user would and confirm the Definition of
  Done in `docs/next-prompt.md` line by line. Do not reason about whether it works.
  Look at it. **Omit this bullet entirely if the project has neither tool** - a check
  that cannot run is worse than no check.

**If any of that fails, stop.** Mark the task `[!]` in `docs/ROADMAP.md`, write what
failed into `docs/PROGRESS.md`, and say so plainly in the chat. Do not mark it `[x]`
and mention the failure in passing. A task that does not build is not done, and the user
cannot read the diff to discover otherwise.

## 2. Record it

1. `docs/PROGRESS.md` - add an entry at the top, matching the format of the entries
   already there. Include the task ID, what changed, the files touched, what you
   actually clicked and saw, and anything you did not finish or were unsure about.
2. `docs/ROADMAP.md` - update the status. `[x]` only if every line of the Definition of
   Done is genuinely true and you watched it be true.
3. `docs/LEARNINGS.md` - add anything that would save time next session: a gotcha, a
   version constraint, a command that works, a dead end worth not repeating. Only if it
   would change a decision later. Skip it otherwise.

## 3. Ship it

`git add -A`, commit with the task ID in the message (`M1-T3: sign-up form`), and push.

Then paste the commit SHA into the PROGRESS entry you just wrote. That short string is
the project's undo button - without it there is no way to say "put it back to before
this task" later.

Do not push if the build failed. A broken deploy is harder to unpick than a broken
working copy.

Finish with a three-line summary: what you built, what you saw when you clicked it,
and what is still open.
```

Three things not to undo later. **Checks before paperwork**, or Claude Code writes an
optimistic entry and then discovers the build is broken. **The push is not optional** -
without it the Vercel deploy never fires and the live URL, the one thing the user can check
from their phone, drifts behind the repo. **The commit SHA** is the difference between
"revert to before M1-T3" being a command and being archaeology.

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
      "Bash(npx tsc --noEmit)",
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

## Superpowers - the plugin that argues with this one

[`obra/superpowers`](https://github.com/obra/superpowers) is a widely used Claude Code plugin, so check whether the user has it before generating anything. It is not a passive library. It is a full development methodology that **loads itself through a SessionStart hook matching `startup|clear|compact`** - so it reloads on every `/clear`, which this workflow tells the user to do between every single task. It is in play on every prompt you hand over, whether or not the prompt mentions it.

**Where it fights this skill.** Its entry skill, `using-superpowers`, says: *"Let's build X" → superpowers:brainstorming first*, and *"Before entering plan mode: if you haven't already brainstormed, invoke the brainstorming skill first."* And `brainstorming` carries a hard gate: do not write any code or scaffold anything until the human has approved a design.

So the user types `/next`, and instead of building the task Cowork already scoped, Claude Code opens a design conversation and asks them to approve decisions that were made here days ago and are written down in `docs/ARCHITECTURE.md`. That is precisely the failure this whole skill exists to prevent: a non-developer being handed choices they have no basis to make. `writing-plans` has the same shape - it wants to author its own plan into `docs/superpowers/plans/`, competing with `docs/next-prompt.md`.

Neither is a bug. Superpowers assumes the person driving Claude Code is a developer who has not planned yet. In this setup the planning already happened, somewhere else, with the person who could not do it alone.

**Where it is a gift.** Three of its skills do things this skill cannot, and they matter more for the user than for a developer:

| Skill | Why it earns its place |
|---|---|
| `verification-before-completion` | *"No completion claims without fresh verification evidence."* Enforced harder than `/checkpoint` can on its own, at exactly the point the user is most exposed. Keep it on. |
| `systematic-debugging` | Root cause before fixes, four phases. Better than the debugging recipe alone. Name it in every Unblock prompt. |
| `requesting-code-review` | Dispatches a reviewer subagent. They cannot review code, so a second model doing it is the only review the work gets besides the plan. |

**How to resolve it.** If the user does not have it, skip this and leave the block out of `CLAUDE.md` - instructions about an absent plugin are noise. If they do: do not uninstall anything, and do not fight it in the prompt. Settle it once in `CLAUDE.md`, which loads alongside superpowers every session, using the block in `doc-pack.md`. It tells Claude Code that brainstorming is already done and points at where the answers live, keeps the three skills above, and redirects plan output to the roadmap. `writing-plans` explicitly honours user preferences on plan location, so this is a supported override rather than a hack.

Then add one line to `docs/next-prompt.md` on every handover, in the Context section:

```
The design work for this task is already done and lives in docs/ARCHITECTURE.md and
docs/DESIGN.md. Do not run brainstorming. If something in this prompt looks wrong,
say so before you build - that is the review, not a fresh design conversation.
```

**Two to leave alone for now.** `using-git-worktrees` and `subagent-driven-development` are real machinery for parallel work across branches. The user builds one task at a time in one branch, so they add moving parts without buying anything, and a stray worktree is a confusing thing to be stuck in. `test-driven-development` is a genuine judgement call: it conflicts with `stack-picker.md`'s advice to defer the test framework, but tests written before the code are the one form of documentation the user can actually read the output of. Leave it off by default, and suggest turning it on when a project reaches the point of having a flow that must not break.

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
