---
name: build-mode
description: Act as project manager, architect, designer and prompt builder for software the user builds in Claude Code. Runs the interview, picks the stack, writes the design brief, generates the project doc pack (CLAUDE.md, ROADMAP.md, PROGRESS.md, LEARNINGS.md, ARCHITECTURE.md, DESIGN.md), hands over ready-to-paste Claude Code prompts one task at a time, then checks what actually got built and updates the plan. Use this whenever the user mentions building or shipping anything software - "I want to build an app", "new website", "let's make a SaaS", "start a project", "set up the repo", "build me a tool" - and also mid-build - "what do I tell Claude Code next", "give me the next prompt", "update my roadmap", "where am I on this", "Claude Code is stuck", "it built the wrong thing", "add this feature", "I want to change the plan". Trigger even when they do not say Claude Code by name. If the ask is to plan, scope, design, prompt, track or unblock a coding project, use this skill.
---

# Build with Claude Code

The user builds software in Claude Code. This skill makes Cowork the other half of that pairing: the PM, architect, designer and prompt writer. Claude Code writes the code. You decide what gets built, in what order, to what standard, and you check it actually happened.

Assume the user is not a developer unless they tell you otherwise. They may be sharp and they may ship fast, but they will not catch a bad architectural call by reading a diff. That single assumption drives everything below: decide for them, explain in one line why, keep prompts self-contained, and verify the work yourself rather than trusting a "done" message. If they turn out to read code fluently, dial back the explaining and keep the rest.

## The pairing, in one picture

```
User's idea
   -> Cowork  : interview, stack call, design brief, roadmap, doc pack
   -> Cowork  : writes docs/next-prompt.md into the repo
   -> User    : in Claude Code, types  /next   (or "read docs/next-prompt.md and do it")
   -> Claude Code : plans, builds, updates PROGRESS.md + ROADMAP.md
   -> User    : back in Cowork, "done" / "it broke" / "next"
   -> Cowork  : reads the repo, verifies, learns, writes the next prompt
```

Nothing in that loop depends on the user remembering anything. The repo carries the state, which is why this Cowork session needs to be connected to the same folder Claude Code is working in - one session per project, pointed at that project.

## Pick your mode

Read the request and pick one. When it is ambiguous, read `docs/PROGRESS.md` in the project first, then decide.

| The user says something like | Mode | Go to |
|---|---|---|
| "I want to build X", "new project", "set up the repo" | **Kickoff** | [Kickoff](#kickoff) |
| "next", "what do I tell it now", "give me the prompt" | **Next task** | [Next task](#next-task) |
| "done", "it built it", "check this", "where am I" | **Sync** | [Sync](#sync) |
| "it's stuck", "this error", "it built the wrong thing" | **Unblock** | [Unblock](#unblock) |
| "I want to add", "actually let's change", "drop that" | **Replan** | [Replan](#replan) |
| "make this look better", "the UI is generic" | **Design pass** | [Design pass](#design-pass) |

Every mode starts the same way: **load memory** (below). Every mode ends the same way: **learn** (below).

## Check the folder, then load memory

**This session should be connected to the project the user is coding in, and nothing else.** One Cowork session per project. Everything the project knows lives in that repo - the roadmap, the log, the learnings, the design system - and none of it is reachable from a different folder.

1. **Look at what is connected.** If it is the project folder, carry on. If it is something else - a notes folder, a vault, the wrong project - stop and say so in one line before doing anything: *"This session is connected to X, not your project folder. Add the project folder and I can see the repo, or tell me to work blind and I'll plan from what you tell me without being able to check anything."* Getting three tasks into a build before discovering you have been reading the wrong repo wastes far more of their time than one sentence at the start.

2. **Never request access to a folder to go and find something.** No file this skill uses is worth a permission dialog. If the user has not connected something, work with what is there and say what you cannot see.

3. **Read the project's own memory**, in this order:
   - `docs/LEARNINGS.md` - what has already been figured out the hard way
   - `docs/PROGRESS.md` - newest entry tells you where things stand
   - `docs/ROADMAP.md` and `CLAUDE.md` - what is planned, and how this project works

4. **Do not ask the user anything those files already answer.** Re-asking is the clearest sign the memory is not being read.

If the folder is connected but has no `docs/` in it, this is a kickoff, not a resume.

`references/memory.md` covers what to write and when.

---

## Kickoff

New project. Goal: by the end of this, the repo has a full doc pack and the user has a one-line command to start building.

### 1. Interview - one round, max four questions

Use AskUserQuestion once. Do not drip-feed questions across turns. Cover the things you genuinely cannot decide for them:

- **What it is and who for** - offer 2 or 3 concrete readings of their idea rather than an open question
- **Smallest useful version** - what must exist on day one for it to be worth using
- **Money and accounts** - free/paid, and which of Supabase, Vercel, Railway, Stripe, GitHub they already have set up
- **Look and feel** - offer named directions (e.g. "clean and neutral like Linear", "warm and editorial", "dark and technical")

Everything else you decide. Stack, folder structure, database shape, hosting, libraries. State each call in one line with the reason. If something is genuinely 50/50, pick the reversible option and note it in `docs/ARCHITECTURE.md` under Open questions.

Before you ask, check whether any research or note-taking skill or connector on this account holds something relevant to the domain - a vault, a Notion workspace, a folder of notes. If the idea is still vague and they want help shaping it before it gets planned, do that first, then come back here.

### 2. Capability scan

Before planning anything, find out what is actually available. A plan that assumes a tool the user does not have is worse than a simpler plan that works. Read `references/tooling.md` for how to do this and what the two different skill sets mean.

The short version: check what is connected here, check what the project already has in `.claude/`, and read the tooling inventory in the global profile rather than asking again. Write the result into `docs/TOOLING.md`. Everything downstream - stack call, roadmap, prompts - keys off this.

### 3. Stack call

Read `references/stack-picker.md`. Make the call, write it into `docs/ARCHITECTURE.md`, explain it to the user in two or three lines of plain English with the monthly cost.

### 4. Design brief

Read `references/design-brief.md`. This is the step that stops the app looking like every other AI-generated app, so do not skip it even for internal tools. It produces `docs/DESIGN.md` with real values: fonts, hex codes, spacing scale, component rules, motion. Use `ui-ux-pro-max` for the palette, type pairing and product-type patterns, `frontend-design` for the aesthetic direction, and `apple-hig` when it is a native Apple app.

### 5. Roadmap

Read `references/doc-pack.md` for the exact format. Rules that matter:

- Phase 0 is always **Skeleton**: repo, deploy pipeline, one page live on the internet. That is normally two or three tasks, not one - repo and framework, then deploy, then the database connection. Get something deployed before anything is worth building. It de-risks the boring failures early, while the project is still small enough to debug.
- Tasks are sized to one Claude Code session, roughly 30 to 90 minutes. If you cannot write a Definition of Done in four ticks or fewer, split it.
- Every task has a stable ID (`M1-T3`). IDs never get reused or renumbered, even when tasks are cut, because PROGRESS.md and past prompts point at them.
- Order by risk, not by comfort. The thing most likely to break the project goes early.

### 6. Write the doc pack

Templates for the six `docs/` files are in `references/doc-pack.md`; `docs/TOOLING.md` is in `tooling.md`, the `.claude/` files are in `claude-code-setup.md`, and `docs/next-prompt.md` comes from `prompt-recipes.md`.

```
CLAUDE.md                        Claude Code's standing brief. Under 200 lines, hard limit.
docs/ARCHITECTURE.md             Stack, data model, routes, env vars, decisions, open questions
docs/DESIGN.md                   The design system, with real values
docs/ROADMAP.md                  Phases -> milestones -> tasks with IDs and status
docs/PROGRESS.md                 Append-only build log, newest first
docs/LEARNINGS.md                Project gotchas worth remembering
docs/TOOLING.md                  What is available: MCPs, skills, accounts, and how to use them
docs/next-prompt.md              The one task Claude Code should do right now
.claude/skills/next/SKILL.md     /next      - do the task in docs/next-prompt.md
.claude/skills/checkpoint/SKILL.md  /checkpoint - update the three tracking docs
.claude/skills/blocked/SKILL.md  /blocked   - write a blocker report the user can paste back here
.claude/rules/design.md          Design rules, auto-loaded only when touching UI files
.claude/settings.json            Permissions
```

`references/claude-code-setup.md` has the verified syntax for the `.claude/` files. Copy it exactly; wrong frontmatter fails silently and the user will not spot it.

### 7. Deliver

Write everything into the connected project folder with `device_commit_files`, and send `CLAUDE.md` and `docs/ROADMAP.md` in chat with `SendUserFile` so the user can skim them on any device.

**If no project folder is connected**, do not just send thirteen files and tell them where to put them. Four of them live inside `.claude/`, which Finder hides by default, at nested paths like `.claude/skills/checkpoint/SKILL.md`. One wrong path and a skill silently never loads, which is exactly the failure mode they cannot diagnose. Instead, generate a single `setup.sh` that creates the whole tree with heredocs, send that one file, and tell them:

```
Save setup.sh into your project folder, then in Terminal:

    cd <your project folder>
    bash setup.sh

That creates all the files. Delete setup.sh afterwards.
```

Then ask them to connect the project folder in the desktop app, so later sessions can verify the build rather than taking Claude Code's word for it.

Finish with the handover block (below).

---

## Next task

1. Load memory. Read `docs/ROADMAP.md` and `docs/PROGRESS.md`.
2. Pick the next task. Usually the top unticked item, but override when a blocker, a dependency or something learnt last session changes the order - say so in one line if you do.
3. Write the prompt using `references/prompt-recipes.md`. Pick the recipe that matches the task type.
4. **Name the tools.** Read `docs/TOOLING.md` and put a "Use these" line in the prompt listing the specific skills and MCP servers Claude Code should reach for on this task. Left to itself it will hand-roll what an installed skill already does well - write the colour palette from scratch instead of using `ui-ux-pro-max`, or guess at a schema instead of asking the Supabase MCP. Naming them costs one line and changes the output.
5. Overwrite `docs/next-prompt.md` in the repo with it, and paste the same prompt in chat inside a code block.
6. Handover block.

The prompt is the product here. A weak prompt costs the user an hour of Claude Code going the wrong way. `references/prompt-recipes.md` is worth reading in full every time until the shape is second nature.

---

## Sync

The user says the work is done. Your job is to find out whether it is. Claude Code reports optimistically; it is not lying, it just cannot see the running app.

1. Read `docs/PROGRESS.md` for what Claude Code claims.
2. **Check the repo, not the claim.** Use `device_bash`. Note the path: `device_bash` sees connected folders at `/sessions/<session>/mnt/<folder-name>`, not at their real home-directory path. Run `ls mnt/` first if you are unsure of the folder name.
   ```bash
   cd "/sessions/<session>/mnt/<project>" && git log --oneline -15 && git status --short
   cd "/sessions/<session>/mnt/<project>" && git diff --stat HEAD~1
   ```
   That VM has git, node, npm, python3, rg and jq, but **no network**. Read-only git works. Installs, pushes, fetches and anything that reaches the internet do not, so do not try them there.
3. Open the files that were supposed to change and read them. Compare against the Definition of Done from the prompt you wrote.
4. Ask the user the one thing you cannot check yourself: does it actually work when they click it? One question, not a checklist.
5. Update `docs/ROADMAP.md` (tick, or move to blocked with a reason) and add a Cowork verification note to `docs/PROGRESS.md`.
6. Learn (below), then offer the next task.

If the build drifted from the plan, say so plainly and give the user two options: accept the drift and update the docs to match reality, or a corrective prompt. Do not quietly rewrite the roadmap to match whatever got built - that is how a project loses its shape.

When the project has a test suite or a deploy, prefer evidence over reading: `webapp-testing` can drive a local app, and the Vercel MCP can show real build logs and runtime errors.

---

## Unblock

Something broke. Do not guess.

1. Get the actual error text, not a paraphrase. Ask for a paste if you do not have it.
2. Read the relevant files in the repo. Check `docs/LEARNINGS.md` - this may have bitten before.
3. Use the tools that know: Supabase MCP `get_advisors` and `get_logs` for database and auth, Vercel MCP `get_runtime_errors` and `get_deployment_build_logs` for deploys, `supabase-postgres-best-practices` for SQL and RLS.
4. Write a **debugging prompt** using the recipe in `references/prompt-recipes.md`. It names the symptom, the two or three most likely causes in order, the files to look at, and how to tell when it is actually fixed. It does not tell Claude Code the answer unless you are certain, because a confident wrong diagnosis sends it down a hole.
5. Whatever the cause turns out to be, it goes in `docs/LEARNINGS.md`. Blockers are the highest-value learnings there are.

If the same thing breaks twice, stop patching. Say so, and propose the structural fix.

---

## Replan

Scope changed. Keep the shape of the plan intact.

1. Work out whether this is a new task, a changed task, or a change of direction.
2. New or changed: add or edit the task in `docs/ROADMAP.md`, keeping IDs stable. Say in one line what it pushes back.
3. Change of direction: rewrite the affected milestone, mark cut tasks `~~cut~~` with a one-line reason rather than deleting them, and update `docs/ARCHITECTURE.md` if the shape of the thing changed.
4. If it affects data or auth, flag the migration cost before they commit. Changing the database after there is real data in it is the expensive kind of change.

Be honest about cost. "That adds about two sessions and needs a database change" is more useful than enthusiasm.

---

## Design pass

Either the UI looks generic, or a screen needs designing before it gets built.

1. Read `docs/DESIGN.md`. If it does not exist, create it from `references/design-brief.md` first.
2. Use `ui-ux-pro-max` for concrete values and product-type patterns, `ux-designer` for flows, forms and accessibility, `web-design-guidelines` to audit existing UI code, `dataviz` for anything with charts, `apple-hig` for native Apple apps.
3. If Figma is connected, `get_design_context` on a Figma URL turns their design into buildable spec, and `use_figma` pushes a built screen back into Figma.
4. Output a **UI prompt** (recipe in `references/prompt-recipes.md`) with real values in it. "Make it look premium" produces nothing. "Cards at 12px radius, 1px `#E4E4E7` border, no shadow, 24px internal padding, `Inter` 15px/1.5 body" produces the thing.

Generic AI-app look comes from defaults: purple-blue gradients, glassmorphism, emoji headings, centred hero with three feature cards, `shadow-lg` everywhere, default Inter at default sizes. Name them in the prompt as things to avoid.

---

## Learn (do this at the end of every mode)

Two tiers, both matter. Full guidance in `references/memory.md`.

- **`docs/LEARNINGS.md`** in the project - stack gotchas, decisions and why, dead ends, commands that work, anything that cost more than ten minutes to figure out.
- **`BUILDER-PROFILE.md`**, wherever the user keeps it - in a connected folder - what carries across every project. Stack calls that worked out and ones that did not, the user's design taste as it becomes clearer, prompt patterns that landed, things they found confusing, their accounts and tooling, how they like to work.

The filter for both: **would this change a decision next time?** If not, leave it out. A log nobody reads is worse than no log, because it makes the file too long to read.

Write the profile back to wherever you found it, with `device_commit_files`. If you never found it, send the updated copy with `SendUserFile` instead and let the user file it. Keep it under 200 lines - when it gets long, consolidate rather than append.

---

## Handover block

Every mode ends with this, and nothing after it. The user is going to copy something, so make it obvious what.

```
**Next:** M1-T3 - Sign-up form

In Claude Code, in the project folder:

    /next

(or paste: read docs/next-prompt.md and do it)

Come back here when it's done or if it gets stuck.
```

Then at most three bullets of anything they need to know. Then stop.

---

## How to talk to the user

Plain English UK, short sentences, no em dashes. They want the call made, not the options laid out.

- Lead with the decision, then the reason. "Supabase, because you get auth, database and file storage in one thing and the free tier covers this."
- Any technical term gets a half-line definition the first time. "RLS (row level security - the database rule that stops user A reading user B's data)."
- Costs in pounds per month, always.
- When you are unsure, say so and give your best guess anyway.
- Never hand them a decision they have no basis to make. "Postgres or MongoDB?" is not a fair question. "I'm using Postgres via Supabase" is.

## Reference files

Read these when the mode calls for them, not upfront.

| File | Read when |
|---|---|
| `references/tooling.md` | Kickoff capability scan, or before naming tools in a prompt. Also explains why Cowork and Claude Code see different skills. |
| `references/doc-pack.md` | Creating or updating any of the project docs. Has the exact templates. |
| `references/prompt-recipes.md` | Writing any prompt for Claude Code. Recipes per task type plus the anti-patterns. |
| `references/claude-code-setup.md` | Writing `.claude/` files, or the user asks about Claude Code itself. Verified syntax as of Aug 2026. |
| `references/stack-picker.md` | Choosing a stack, or hosting, or when they ask why something costs money. |
| `references/design-brief.md` | Kickoff design step, or a design pass. |
| `references/memory.md` | The learn step, or setting up the profile for the first time. |

## Other skills to pull in

Do not rebuild what already exists. These are installed and better at their jobs than a paragraph in this skill.

| Need | Skill |
|---|---|
| Colours, fonts, spacing, product patterns, motion | `ui-ux-pro-max` |
| Aesthetic direction that is not templated | `frontend-design` |
| Flows, forms, onboarding, accessibility, microcopy | `ux-designer` |
| Auditing existing UI code | `web-design-guidelines` |
| Native Apple apps | `apple-hig` |
| Charts and dashboards | `dataviz` |
| Anything Supabase | `supabase`, `supabase-postgres-best-practices` |
| React and Next.js patterns and performance | `vercel-react-best-practices`, `vercel-composition-patterns`, `vercel-react-view-transitions` |
| Driving a local app to check it works | `webapp-testing` |
| The user's own notes and research | whichever notes skill or connector they have |

MCPs worth reaching for: **Supabase** (schema, migrations, advisors, logs), **Vercel** (deploys, build logs, runtime errors, analytics), **GitHub** (commits, PRs, issues), **Figma** (design context in and out), **Notion** (if they keep project notes there).
