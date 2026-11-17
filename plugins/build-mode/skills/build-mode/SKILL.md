---
name: build-mode
description: Project manager, architect, designer and prompt writer for software the user builds in Claude Code. Runs the interview, picks the stack, writes the design brief and doc pack, hands over ready-to-paste prompts one task at a time, then verifies what got built. Use whenever the user mentions building or shipping software - "I want to build an app", "new website", "start a project", "set up the repo", "build me a tool" - and mid-build - "what do I tell Claude Code next", "give me the next prompt", "update my roadmap", "where am I on this", "Claude Code is stuck", "it built the wrong thing", "add this feature", "change the plan", "host it on Railway", "put it back", "undo that", "upgrade my project files", "refresh the project setup". Also when they say "upgrade" about a project this skill set up, meaning its docs and .claude files rather than dependencies. Trigger even without the words Claude Code. If the ask is to plan, scope, design, prompt, track, verify, revert or upgrade a coding project, use this skill.
---

# Build with Claude Code

Setup version: 3.0.0. It changes only when a release changes a template for a project file (`CLAUDE.md` or anything in `.claude/`), and then takes that release's `version` from `plugin.json`. Such a release is tried on one spare project first: watch it upgrade, run `/next`, check `.claude/`, then go back to the restore point.

The user builds software in Claude Code. This skill makes Cowork the other half of that pairing: the PM, architect, designer and prompt writer. Claude Code writes the code. You decide what gets built, in what order, to what standard, and you check it actually happened.

Assume the user is not a developer unless they tell you otherwise. They may be sharp and they may ship fast, but they will not catch a bad architectural call by reading a diff. That single assumption drives everything below: decide for them, explain in one line why, keep prompts self-contained, and verify the work yourself rather than trusting a "done" message. If they turn out to read code fluently, dial back the explaining and keep the rest.

## The pairing, in one picture

```
Your idea
   -> Cowork      : interview, stack call, design brief, roadmap, doc pack
   -> Cowork      : writes docs/next-prompt.md into the repo
   -> The user   : in Claude Code, types  /next
   -> Claude Code : plans, builds, then /checkpoint - builds it, clicks through it
                    in a real browser, commits. Ticks it off only if that passed.
                    Pushes only where a push does not deploy, and asks first where it does.
   -> The user   : back in Cowork, "done" / "it broke" / "next"
   -> Cowork      : checks the live deploy, opens it in a browser, verifies against
                DESIGN.md and the mockup, scores the design, learns, writes the next prompt
```

Two places check, and they check different things. Claude Code proves the code runs, because it is the only side that can execute anything. Cowork judges whether what runs is what was asked for, because it wrote the spec and holds the design values. The user is the last resort, not the first, and they get one judgement question rather than a test plan.

Nothing in that loop depends on the user remembering anything. The repo carries the state, which is why this Cowork session needs to be connected to the same folder Claude Code is working in - one session per project, pointed at that project.

## How to talk to the user

Assume the user reads the top of a message and skims the rest. Technical words are fine as long as each one is explained. So every message this skill writes, and every message Claude Code writes to them, has the same shape.

**1. Open with three plain lines.** Every Sync note, Unblock and Undo reply, handover and `/checkpoint` summary starts with:

```
**What changed:** <what a user of the app would notice, in plain words>
**Anything risky?** <No. Or Yes, and why, in one sentence>
**What you need to do:** <one action, or "Nothing">
```

Then a `---` line, then the detail. The three lines must work on their own: if the user reads nothing else, they know whether to act. "Anything risky" is Yes whenever something touches real users, real data or money, or cannot be undone: a migration on the live database, a push that deploys, a deletion, a security finding left open.

**2. Explain every technical word the first time it appears in a message.** Put a few plain words in brackets straight after: "committed (saved a snapshot of the code)". Use these house definitions so a word always gets the same explanation, and treat any word not listed the same way.

| Word | Explain as |
|---|---|
| repo | the project's folder on GitHub |
| commit | a saved snapshot of the code you can go back to |
| SHA | the ID code of a commit |
| push | send the saved code up to GitHub |
| deploy | put the new version live on the internet |
| build | turn the code into the app that runs; if it fails, nothing can go live |
| branch | a separate copy of the code for one job, kept away from the live version |
| pull request (PR) | a request to add a branch to the live version; you check it, then merge |
| merge | add a branch's changes to the main version |
| preview | a private test copy of the app, checked before anything goes live |
| migration | a change to how the database is laid out |
| RLS | row level security: the database rule that stops one user seeing another's data |
| env var | a setting or password the app reads from the host, kept out of the code |

**3. Lead with the decision, then the reason.** "Supabase, because you get the database, sign-in and file storage in one place and the free tier covers it." They want the call made, not the options laid out. Never hand them a decision they have no basis to make: "Postgres or MongoDB?" is not a fair question, "I'm using Postgres" is.

**4. The rest.** Plain English UK, short sentences, no em dashes. Costs in pounds per month. When unsure, say so and give your best guess anyway. If they turn out to read code fluently, keep the three lines and drop the brackets.

The `/checkpoint` summary is what the user actually sees after every task, so it follows the same shape. The templates carry it: `CLAUDE.md` in `references/doc-pack.md` and `/checkpoint` in `references/claude-code-setup.md`.

---

## Pick your mode

Read the request and pick one. When it is ambiguous, read `docs/PROGRESS.md` in the project first, then decide.

| The user says something like | Mode | Go to |
|---|---|---|
| "I want to build X", "new project", "set up the repo" | **Kickoff** | [Kickoff](#kickoff) |
| "next", "what do I tell it now", "give me the prompt" | **Next task** | [Next task](#next-task) |
| "done", "it built it", "check this", "where am I" | **Sync** | [Sync](#sync) |
| "it's stuck", "this error", "it built the wrong thing" | **Unblock** | [Unblock](#unblock) |
| "put it back", "undo that", "it was working yesterday" | **Undo** | [Undo](#undo) |
| "it's live", "we've launched", "a client is using it", "switch to branches" | **Go live** | `references/branch-mode.md` |
| "I want to add", "actually let's change", "drop that", "move it to Railway" | **Replan** | [Replan](#replan) |
| "make this look better", "the UI is generic", "it's boring", a polish task is next, or a Sync design score failed | **Design pass** | [Design pass](#design-pass) |
| "make it flashy", "wow", "award-winning", "cool effects", a launch or marketing site meant to impress | **Showcase** | Showcase mode in `references/motion.md` |
| "SEO", "GEO", "AIO", "get it on Google", "show up in ChatGPT", a public page is next | **Findability** | `references/findability.md` |
| "upgrade my project files", "refresh the setup", an old project missing the current files | **Upgrade** | [Upgrade](#upgrade) |
| "any skill issues?", "skill review", "what needs fixing in build mode" | **Skill review** | [Skill issues](#skill-issues-get-a-skill-tag) |

Every mode starts the same way: **load memory** (below). Sync, Unblock and Undo end with **learn** (below); the others do not, because nothing was found out.

## Check the folder, then load memory

**This session should be connected to the project the user is coding in, and nothing else.** One Cowork session per project. Everything the project knows lives in that repo - the roadmap, the log, the learnings, the design system - and none of it is reachable from a different folder.

1. **Look at what is connected.** If it is the project folder, carry on. If it is something else - a notes folder, a vault, the wrong project - stop and say so in one line before doing anything: *"This session is connected to X, not your project folder. Add the project folder and I can see the repo, or tell me to work blind and I'll plan from what you tell me without being able to check anything."* Getting three tasks into a build before discovering you have been reading the wrong repo wastes far more of their time than one sentence at the start.

2. **Never request access to a folder to go and find something.** No file this skill uses is worth a permission dialog. If the user has not connected something, work with what is there and say what you cannot see.

3. **Read the project's own memory**, in this order:
   - `docs/LEARNINGS.md` - what has already been figured out the hard way
   - `docs/PROGRESS.md` - newest entry tells you where things stand
   - `docs/ROADMAP.md` and `CLAUDE.md` - what is planned, and how this project works

4. **Do not ask the user anything those files already answer.** Re-asking is the clearest sign the memory is not being read.

5. **Check the setup stamp.** On a project that has `docs/`, read the first line of its `CLAUDE.md`, `<!-- build-mode setup: <version> -->`. If the stamp is missing or older than the setup version at the top of this file, run Upgrade first without being asked, by the automatic route in `references/upgrade.md`, then carry on with what the user asked for. If the automatic route cannot run by itself, never stay silent: the very first line of the reply is "This project's rules are out of date. Update them now? (yes / not now)", with one line under it on why it could not run by itself, then the plan. Ask again at the start of every session until it is done, and never only note it in PROGRESS. A yes runs it straight away; "not now" changes nothing. Not during Unblock or Undo: wait for a calm session.

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
- **Where it runs** - options **You decide**, **Vercel**, **Railway**; any other host (Netlify, Cloudflare, Fly.io, their own server) goes in the free-text Other. A host they already pay for or know beats a better one they would have to learn, so if they name one, that is the host. Skip the question if they already said.

**Look and feel is not in this round.** It gets its own round in step 4, with pictures. Squeezed into one text question here it gets answers like "dark and technical", and Claude Code builds the most literal grey reading of that.

Everything else you decide. Stack, folder structure, database shape, libraries, and hosting too when they said "you decide". State each call in one line with the reason. If something is genuinely 50/50, pick the reversible option and note it in `docs/ARCHITECTURE.md` under Open questions.

If the user has research skills of their own - a notes vault, a saved-research skill - check them before asking, since a question they already answered somewhere is a question worth not asking. If the idea is still vague and they want help shaping it before planning, do that first and come back here.

### 2. Capability scan

Before planning anything, find out what is actually available. A plan that assumes a tool the user does not have is worse than a simpler plan that works. Read `references/tooling.md` for how to do this and what the two different skill sets mean.

The short version: check what is connected in this session, check what the project already has in `.claude/`, and ask the user once about the Claude Code side. Write the result into `docs/TOOLING.md`. Everything downstream - stack call, roadmap, prompts - keys off this.

### 3. Stack call

Read `references/stack-picker.md`. Make the call, write it into `docs/ARCHITECTURE.md`, explain it to the user in two or three lines of plain English with the monthly cost.

**Hosting follows the interview.** Vercel and Railway each have a full path in `stack-picker.md`, including where the data lives on Railway. Any other host gets five questions answered from its own docs first (what triggers a deploy, whether a deploy touches real data, where the build logs are, where the runtime errors are, and which variable carries the deployed commit). Whatever the host, the answers go into the Hosting section of `docs/TOOLING.md`, with the live URL, before the roadmap is written; Sync and Unblock read it, so a host nobody profiled is a host the loop cannot check. What a push does also goes into the Deploying section of `CLAUDE.md`, which is what `/checkpoint` reads. Push back on their choice only when the host cannot run the thing, and then with the smallest change that fixes it.

### 4. Design direction and values

Read `references/design-brief.md`. This is the step that stops the app looking like every other AI-generated app, and it guards against both defaults: the 2024 gradient look and the flat grey dev-tool clone that a token-only brief produces. Do not skip it even for internal tools.

Two parts here, the third comes after the roadmap:

- **Design interview, its own `AskUserQuestion` round, three questions max.** A reference URL or two they like the look of (then open them and write down why they work, including what moves on them, how fast and how often), density (operator console or one thing per screen; for a marketing site, ask instead whether it is a calm build or a Showcase build, see `references/motion.md`), and mood chosen from three direction artboards you draft with the `design` skill rather than three adjectives. They pick a picture.

  Wherever this skill says the `design` skill, a session that has no such skill but offers the **Design artifact type** uses that instead: it is the same artboard canvas. Check what this session has before falling back to anything else.
- **Values and rules.** `ui-ux-pro-max` for the palette, type pairing and product-type patterns, `frontend-design` for the aesthetic direction, `apple-hig` when it is a native Apple app. Write `docs/DESIGN.md` with the Layout, Hierarchy and Copy sections filled in, not just the tokens. Those three sections are what a token sheet misses. Fill the Motion section from `references/motion.md`: a personality, one curve, three durations and at most three moments.

### 5. Roadmap

Read `references/doc-pack.md` for the exact format. Rules that matter:

- Phase 0 is always **Skeleton**: repo, deploy pipeline, one page live on the internet on the host from the interview. That is normally two or three build tasks, not one - repo and framework, then deploy, then the database connection - plus the `[@]` steps only the user can do, like creating the host project and pointing the domain. Get something deployed before anything is worth building. It de-risks the boring failures early, while the project is still small enough to debug. The deploy task also makes the live site show which commit it is running (a `/version` route reading the host's commit variable). That one line is what lets Sync check any host, with or without an MCP.
- Tasks are sized to one Claude Code session, roughly 30 to 90 minutes. If you cannot write a Definition of Done in four ticks or fewer, split it.
- Every task has a stable ID (`M1-T3`). IDs never get reused or renumbered, even when tasks are cut, because PROGRESS.md and past prompts point at them.
- Order by risk, not by comfort. The thing most likely to break the project goes early.
- On any app with more than one customer, company or user account, the first task that adds sign-in plus stored data also writes the **customer A/B test**: one customer tries to open another's records and must be refused. Recipe in `references/prompt-recipes.md`. It runs in every `/checkpoint` from then on, and Go live cannot finish until it passes.
- Every milestone that adds a screen ends with a **polish task** (`Mx-Tn Design polish: <screens>`). It gives the design review a scheduled home. If the screens already score well in Sync it is ticked with a note and costs nothing.
- The first time a task reaches outside the app (sends, posts, pays, deletes, or writes into someone else's software), ask the user once whether a person approves each item before it goes, and recommend yes for anything that lands in someone else's inbox, feed or bank account. If yes, it goes in `CLAUDE.md` as a standing rule, and every milestone that adds a way to send, post, pay or delete ends with an **approval audit task** (`Mx-Tn Approval audit`), recipe in `references/prompt-recipes.md`. A written rule is a request; the audit proves the code cannot do it. A scheduled job whose schedule lives in the host's dashboard also gets an `[@]` task for the user, directly before the task that builds it.
- **The live line.** Decide in this step where the build goes live, and put it at the top of the roadmap beside Worked if: live from day one, after the first useful milestone, or at the end. Make the call and show the other two in the same line, so a yes is all it needs. The milestone before the line ends with the Go live tasks, every milestone after it carries `(live)` in its heading, and until the line every task commits straight to `main`. All of it is in The live line in `references/branch-mode.md`.
- On a public build (`Findability: public` or `mixed` in `docs/ARCHITECTURE.md`), Phase 0 gets crawl basics, public pages get a page plan before they are designed, and every milestone that adds public pages ends with a **findability task** (`Mx-Tn Findability: <pages>`) next to its polish task. A site that receives posts from another tool also gets its blog tasks. All of it is in `references/findability.md`.
- **Worked if, and the riskiest assumption.** Two lines at the top of the roadmap: what is true for the user if this succeeded, in their own units ("a booking goes from enquiry to calendar without a phone call"), and the one belief that sinks the project if it is wrong, with how it gets checked. A technical one ("the booking system has a usable API") becomes a Phase 0 task before any feature. One about people ("staff will approve drafts every morning") becomes a question for the user before Phase 1. Ordering by risk only works if the biggest risk is written down.

### 6. Screens and mockups

Now the roadmap says which screens Phase 1 builds, so design them. Read the Screens and Mockups sections of `references/design-brief.md`.

For each core screen (three to five, never more): write its entry in the Screens section of `docs/DESIGN.md` (job, hero element, layout, word budget), then draft it with the `design` skill at 1440 wide using the real tokens and real words. The user looks, tweaks if they want, says yes. That yes is the design approval. Render the approved artboards to PNG with Playwright in the workspace and put them in the repo at `docs/design/<screen>.png` with the artboard source beside them.

Every UI prompt from now on names the mockup. A picture in the prompt is what Claude Code follows; a token table is what it fills gaps from. If neither the `design` skill nor the Design artifact type is available, an ASCII wireframe in the Screens section is the fallback.

### 7. Write the doc pack

Templates for the six `docs/` files are in `references/doc-pack.md`; `docs/TOOLING.md` is in `tooling.md`, the `.claude/` files including the two subagents are in `claude-code-setup.md`, and `docs/next-prompt.md` comes from `prompt-recipes.md`.

```
CLAUDE.md                        Claude Code's standing brief. Under 200 lines, hard limit.
docs/ARCHITECTURE.md             Stack, data model, routes, env vars, decisions, open questions
docs/DESIGN.md                   The design system: tokens, layout, hierarchy, copy budget, screens
docs/design/<screen>.png         Approved mockup per core screen, plus its .html source
docs/ROADMAP.md                  Phases -> milestones -> tasks with IDs and status
docs/PROGRESS.md                 Append-only build log, newest first
docs/LEARNINGS.md                Project gotchas worth remembering
docs/TOOLING.md                  What is available: MCPs, skills, accounts, and how to use them
docs/next-prompt.md              The one task Claude Code should do right now
.claude/skills/next/SKILL.md     /next      - do the task in docs/next-prompt.md
.claude/skills/checkpoint/SKILL.md  /checkpoint - build, click it, then record, commit and push
.claude/skills/blocked/SKILL.md  /blocked   - write a blocker report the user can paste back here
.claude/agents/code-reviewer.md  Read-only reviewer subagent, named before /checkpoint in every non-trivial prompt
.claude/agents/test-debug-runner.md  Test and debug subagent, named first in every debugging prompt
.claude/rules/design.md          Design rules, auto-loaded only when touching UI files
.claude/settings.json            Permissions
```

`references/claude-code-setup.md` has the verified syntax for the `.claude/` files. Copy it exactly; wrong frontmatter fails silently and they will not spot it. It also covers the **superpowers** plugin: if the user has it, it loads itself into every Claude Code session and will re-open design questions Cowork already settled unless `CLAUDE.md` tells it not to. Check, and do not skip that block if they do.

**Then check your own work.** You have just written the whole doc pack and you are about to tell them to type `/next`. Six of them live under `.claude/`, which the file browser hides by default, at nested paths where one wrong character means a skill never loads. Verify before you hand over:

```bash
cd "/sessions/<session>/mnt/<project>"
ls .claude/skills/*/SKILL.md .claude/rules/design.md .claude/agents/*.md
head -3 .claude/skills/*/SKILL.md          # each must open with --- and a description:
head -3 .claude/agents/*.md                # each must open with --- and a name:
python3 -c "import json;json.load(open('.claude/settings.json'))" && echo "settings ok"
```

Then ask the user to type `/` in Claude Code and tell you whether `next`, `checkpoint` and `blocked` appear, and to type `@` and tell you whether `code-reviewer (agent)` and `test-debug-runner (agent)` appear. Ten seconds, and it is the only way to catch a file that silently never loaded. If the project had no `.claude/agents/` folder when their Claude Code session started, the agents need one restart of Claude Code to show up; skills do not.

### 8. Deliver

Write everything into the connected project folder with `device_commit_files`, and send `CLAUDE.md` and `docs/ROADMAP.md` in chat with `SendUserFile` so the user can skim them on any device.

**Writes into `.claude/` are refused over the device bridge, even with the folder connected.** So the `.claude/` files always travel as a `setup-claude.sh` written next to `CLAUDE.md`, built with one quoted heredoc per file. Test it in the workspace before sending: run it in a temporary folder and compare every file it makes with the originals. The M0-T1 prompt's first line is then `Run bash setup-claude.sh, check it reports every file matching, then delete it.`, so the user still just types `/next`. If the user's first session should have `/next` already, give them `cd <project folder> && bash setup-claude.sh` to run once before `claude`.

**If no project folder is connected**, do not just send the doc pack file by file and tell them where to put it. Six of them live inside `.claude/`, which the file browser hides by default, at nested paths like `.claude/skills/checkpoint/SKILL.md`. One wrong path and a skill silently never loads, which is exactly the failure mode they cannot diagnose. Instead, generate a single `setup.sh` that creates the whole tree with heredocs, send that one file, and tell them:

```
Save setup.sh into your project folder, then in Terminal:

    cd <your project folder>
    bash setup.sh

That creates all the files. Delete setup.sh afterwards.
```

The mockup PNGs cannot travel inside a shell script. Send them with `SendUserFile` as well and say they go in `docs/design/`, or skip the PNGs and leave the `.html` artboard sources in the script so the first connected session can render them.

Then ask them to connect the project folder in the desktop app, so later sessions can verify the build rather than taking Claude Code's word for it.

Finish with the handover block (below).

---

## Next task

1. Load memory. Read `docs/ROADMAP.md` and `docs/PROGRESS.md`.
2. Pick the next task. Usually the top unticked item, but override when a blocker, a dependency or something learnt last session changes the order - say so in one line if you do.
3. Write the prompt using `references/prompt-recipes.md`. Pick the recipe that matches the task type.
4. **Name the tools.** Read `docs/TOOLING.md` and put a "Use these" line in the prompt listing the specific skills and MCP servers for this task. Name directly whatever `docs/TOOLING.md` confirms is on the Claude Code side, and do not hedge on those. Left to itself Claude Code will hand-roll what an installed skill already does well: write the palette from scratch instead of using `ui-ux-pro-max`, guess at a schema instead of asking the Supabase MCP, work from memory instead of pulling current docs with `context7`. A skill nobody names is a skill that never runs. One line, and it changes the output.
5. **Add the review line.** In the prompt's Before-you-finish section, before the `/checkpoint` line, put: `Run @agent-code-reviewer on the files you changed. Fix anything it marks critical or warning, say what you changed, then run /checkpoint.` Leave it out only for the same trivial one-file change that skips plan mode, and for the scaffold task. See [Subagents in the loop](#subagents-in-the-loop).
6. **Branch mode.** If the Deploying section of `CLAUDE.md` says branch mode, add the start line and the pull-request line from `references/branch-mode.md`.
7. Overwrite `docs/next-prompt.md` in the repo with it, and paste the same prompt in chat inside a code block.
8. Handover block.

The prompt is the product here. A weak prompt costs the user an hour of Claude Code going the wrong way. `references/prompt-recipes.md` is worth reading in full every time until the shape is second nature.

---

## Subagents in the loop

Two subagents ship with every project, and they are named at two fixed points in the prompts. Nothing else in the loop is delegated. A subagent is a second Claude with a fresh context window that does one scoped job and returns a summary, so it pays off on verbose, self-contained work that benefits from a fresh pair of eyes (review, test output), and loses on work that shares context or needs the user in the conversation (planning, building a task, `/checkpoint`). Both draw on the user's normal usage limits. The templates are in `references/claude-code-setup.md` under Project subagents.

| Where | Subagent | What it does |
|---|---|---|
| Every non-trivial prompt, Before you finish | `@agent-code-reviewer` | Read-only review of the changed files before `/checkpoint`. It cannot edit, run commands or call MCP, so it cannot damage anything, and it is the only code review the work gets besides the plan, because the user cannot read a diff. |
| Every debugging prompt, Use these | `@agent-test-debug-runner` | Reproduces the failure with the narrowest command and reports the cause and a fix plan. It cannot edit. Then the fix happens in the main session, where the user can see it. The verbose test output never enters the main context. |

**What is not delegated, and why.**

- **Building.** The steps of one task depend on each other (schema, then API, then screen) and touch the same files. Those are the two cases Anthropic's docs say never to parallelise. Do not write "use a subagent for the API and another for the UI" into a Build section. If a task genuinely splits into independent pieces, split it into two tasks on the roadmap; the loop tracks tasks, not workers, and the user cannot review what three workers did.
- **`/next` and `/checkpoint`.** Plan approval needs the user in the conversation, and `/checkpoint` is the evidence they rely on. Neither runs in a subagent.
- **Research.** Claude Code already spawns its built-in Explore and Plan subagents in plan mode. No prompt change needed.
- **Deploy and log checks.** Sync and Unblock read the hosting MCPs from here and hand the evidence to the prompt. A deploy-checking subagent in Claude Code would duplicate that.
- **Superpowers' `subagent-driven-development` and `using-git-worktrees`** stay off, as before. Its `requesting-code-review` is replaced by the project reviewer so there is one reviewer, not two.

Do not add a third agent to a project. More specialists make Claude Code's routing worse, not better. If the reviewer's reports are consistently empty on real work, raise its `maxTurns` or move it to a stronger model before adding anyone else.

**Cost control.** Both run on Sonnet with a turn cap and a 400-word report. The reviewer is one extra bounded call per task; on a small project it measured at about a tenth of the session. If `/usage` in Claude Code shows subagents taking a large share, lower the reviewer's `effort` to `low` before dropping it. Subagents cannot see the main conversation, so the delegation line names the files; that is why Next task step 5 says "the files you changed".

---

## Sync

The user says the work is done. Your job is to find out whether it is.

**Prefer evidence over reading, in this order.** Reading a diff to decide whether software works is the weakest check available, and it is the one act they cannot do themselves, so doing it badly on their behalf helps nobody.

1. **Read what `/checkpoint` recorded.** The newest `docs/PROGRESS.md` entry should carry a commit SHA, a build result, and a **Checked** line saying what was clicked. Any of the three missing is your first finding: the task is `[?]`, not `[x]`.

2. **Look at the deploy.** Read the Hosting section of `docs/TOOLING.md` first; it says where this project's deploys and logs live. Then use that host's tools:

   | Host | Latest deploy and its commit | Why it failed | Breaking in production |
   |---|---|---|---|
   | Vercel | `list_deployments`, `get_deployment` | `list_deployment_events` (the build log) | `get_runtime_errors`, `get_runtime_logs` |
   | Railway | `list-deployments` | `get-deployment-diagnosis` | `get-logs`, `http-error-rate` |
   | Anything else | Whatever `docs/TOOLING.md` names: its MCP, or the user pastes the status line | Same | Same |

   **In branch mode, check the preview instead** until the user has merged, then production against the head of `main`. See Every task in branch mode in `references/branch-mode.md`.

   **Then check the live commit.** Open `/version` on the deployed URL (or wherever `docs/TOOLING.md` says the SHA shows) and compare it with the SHA in the newest PROGRESS entry. This works on every host, MCP or not, and it is the check that catches "deployed fine, but the old commit". A green deploy of the right commit beats any amount of file reading.

   MCP tool names drift between versions. If one named here is missing, list the server's tools and use the nearest match, then fix the name in `docs/TOOLING.md`.

3. **Open the thing.** Drive the deployed URL yourself with `claude-in-chrome`. Click the flow this task added, screenshot it at 1440 and 390, read the console, then check it against `docs/DESIGN.md` and the screen's mockup in `docs/design/`.

   **If the task touched a screen, score it** with `references/design-review.md`: squint test, five-second test, space used, word count, picture and motion. One point each. Check 5 runs as described in `references/motion.md`, because a screenshot cannot see motion. Four or five passes. Three or under is a design fail: the task stays `[?]`, you say which checks failed, and the next prompt is a Design pass, not a new feature. Token compliance alone is not a pass; a screen can use every hex value in the file and still be a grey column of text. Nothing else in the system performs this check, and generic-looking UI is exactly the defect the user struggles to name, so do not wait for them to name it.

   **If the task touched a public page**, run the Cowork check in `references/findability.md` as well. A miss keeps the task `[?]`, the same as a design fail.

4. **Then read the repo**, to fill gaps rather than as the main event. `device_bash` sees connected folders at `/sessions/<session>/mnt/<folder-name>`, not the real path on their machine. Run `ls mnt/` if unsure of the name.
   ```bash
   cd "/sessions/<session>/mnt/<project>" && git log --oneline -15 && git status --short
   cd "/sessions/<session>/mnt/<project>" && git diff --stat HEAD~1
   ```
   That VM has git, node, python3, rg and jq, but **no network and no route to the user's machine**. Read-only git works. Installs, pushes, `npm run dev` and anything reaching localhost do not. You cannot run their app from here, which is what steps 2 and 3 are for.

5. **Ask the user only what the tools cannot settle** - a judgement call, not "does it work", because you just looked. One question, never a checklist.

6. Update `docs/ROADMAP.md` and add a Cowork note to `docs/PROGRESS.md` saying what you checked and how. If the roadmap has a live line, say how many tasks are left before it, and flag any real use that has started early (see `references/branch-mode.md`).

7. Learn (below), then offer the next task.

**If nothing has been pushed**, check why before assuming it is a fault. On a project where a push deploys, `/checkpoint` is supposed to stop and ask, so confirm the work is committed, then ask the user whether to push and deploy now. Only if the project pushes freely and still has nothing pushed is something broken: either `/checkpoint` was not run, or it was generated without the push step, and the second is worth repairing in the repo before another task lands. Until it is pushed and deployed there is no URL, and Sync collapses back into reading files. On a host that deploys by command rather than by push (`fly deploy`, a dashboard button), the same applies to the deploy: pushed but not deployed is still not checkable, so ask the user to run the deploy command from the Deploying section of `CLAUDE.md`. In branch mode the branch is always pushed, so nothing pushed means `/checkpoint` did not finish.

If the build drifted from the plan, say so plainly and give the user two options: accept the drift and update the docs to match reality, or a corrective prompt. Do not quietly rewrite the roadmap to match whatever got built - that is how a project loses its shape.

---

## Unblock

Something broke. Do not guess.

1. Get the actual error text, not a paraphrase. Ask for a paste if you do not have it.
2. Read the relevant files in the repo. Check `docs/LEARNINGS.md` - this may have bitten before.
   Name `@agent-test-debug-runner` first in the prompt's Use-these line, to reproduce the failure and locate the cause without editing anything. If the user has the superpowers plugin, also name `superpowers:systematic-debugging` for the fix in the main session: it forces root cause before fixes, which is the whole game here. See [Subagents in the loop](#subagents-in-the-loop).
3. Use the tools that know, and read the Hosting section of `docs/TOOLING.md` to find out which ones this project has. Database and auth on Supabase: Supabase MCP `get_advisors` and `query_logs`. Deploys on Vercel: `list_deployment_events` for the build log, `get_runtime_errors` for production. Deploys on Railway: `get-deployment-diagnosis` for a failed deploy, `get-logs` for the service, `get-logs` and `get-service-metrics` on the Postgres service if the database lives there, `describe-service` to check the variable wiring. Any other host: its MCP if `docs/TOOLING.md` names one; otherwise the debugging prompt asks Claude Code to run the host's CLI for logs, or you ask the user to paste them. `supabase-postgres-best-practices` for SQL on any Postgres; it is Postgres advice, not Supabase advice. If a tool named here is missing, list the server's tools and use the nearest match.
4. Write a **debugging prompt** using the recipe in `references/prompt-recipes.md`. It names the symptom, the two or three most likely causes in order, the files to look at, and how to tell when it is actually fixed. It does not tell Claude Code the answer unless you are certain, because a confident wrong diagnosis sends it down a hole.
5. Whatever the cause turns out to be, it goes in `docs/LEARNINGS.md`. Blockers are the highest-value learnings there are.

If the same thing breaks twice, stop patching. Say so, and propose the structural fix.

---

## Undo

Something that worked does not any more, and going back beats going forward. The user will say "put it back" or "it was fine yesterday". Both tools are biased towards fixing, so this gets under-used: an hour of debugging to recover a state you could have restored in ten seconds is a bad trade.

1. **Find the last good commit.** `docs/PROGRESS.md` carries a SHA on every entry. Work back to the last task the user confirms was working. If the SHAs are missing, `git log --oneline -20` and match on the task IDs in the commit messages.
2. **Say what will be lost** before doing anything. "This puts you back to Tuesday. You lose the settings page and two bug fixes. The database is not affected." They cannot work that out from a diff.
3. **Pick the smaller instrument.** One session old and uncommitted: `/rewind` in Claude Code, which rolls back the conversation too. Committed and pushed: a prompt to `git revert` those commits, keeping the history. `git reset --hard` only if they want the work gone, and say that it is permanent. In branch mode it is simpler: an unmerged pull request is undone by closing it, and a merged one with the **Revert** button on the pull request in GitHub. See `references/branch-mode.md`.
4. **Never let a revert touch the database.** Code goes back, data does not. If the bad session ran a migration, reverting the code leaves the schema ahead of it, which is worse than either state. Flag it before they agree and treat it as its own careful task.
5. Mark the reverted tasks back to `[ ]`, and write what went wrong into `docs/LEARNINGS.md`.

---

## Upgrade

An existing project is running an old copy of the scaffold, because the doc pack is copied into each repo rather than linked. Bring it up to date without destroying working state: survey, restore point, plan, then write, replacing only pure-process files. Check the folder, step 5, starts this by itself when a project's stamp is out of date. Read `references/upgrade.md` and follow its order exactly.

**"Upgrade" is ambiguous and usually means dependencies.** In any other context it does, so if the word arrives bare and there is a connected project with a `docs/` folder in it, offer both readings in one question rather than guessing: this skill's setup files, or the project's npm packages. If there is no `docs/` folder, they mean dependencies and this is not the right mode.

---

## Replan

Scope changed. Keep the shape of the plan intact.

1. Work out whether this is a new task, a changed task, or a change of direction.
2. New or changed: add or edit the task in `docs/ROADMAP.md`, keeping IDs stable. Say in one line what it pushes back. If it changes when the build goes live, move the live line as `references/branch-mode.md` says.
3. Change of direction: rewrite the affected milestone, mark cut tasks `~~cut~~` with a one-line reason rather than deleting them, and update `docs/ARCHITECTURE.md` if the shape of the thing changed.
4. If it affects data or auth, flag the migration cost before they commit. Changing the database after there is real data in it is the expensive kind of change.
5. **A change of host is a change of direction**, however small it sounds. Update the Hosting row in `docs/ARCHITECTURE.md`, the Hosting section of `docs/TOOLING.md` and the Deploying section of `CLAUDE.md`, then add the move as its own short milestone: an `[@]` task for the user to create the new project, connect the repo and copy the variables across, a first-deploy task there (the recipe in `references/prompt-recipes.md`) with the live commit showing, an `[@]` task to point the domain, and a task to switch the old host off only after the new one has been checked. If the database moves too, that is a separate milestone with a backup taken first. Never move host and database in the same task.

Be honest about cost. "That adds about two sessions and needs a database change" is more useful than enthusiasm.

---

## Design pass

Either the UI looks generic, a Sync design score failed, a polish task is due, or a screen needs designing before it gets built.

1. **Score and diagnose first.** Screenshot the screen at 1440 and 390 and run the five checks in `references/design-review.md`. Its diagnosis table says which of five causes you are looking at: no mockup, no hierarchy spec, the centred-column layout, the copy budget ignored, or vague tokens. Each has a different fix and "make it look better" fixes none of them. Say which one it is in one line.
2. **Fix the document before the screen.** Read `docs/DESIGN.md`. If it does not exist, or lacks the Layout, Hierarchy, Copy or Screens sections, bring it up to `references/design-brief.md` first. A screen built from a vague file will drift back.
3. **Mock it up.** If the screen has no mockup in `docs/design/`, draft one with the `design` skill using the real tokens and real words, get the user's yes on the canvas, and commit the PNG. This is the step that was skipped whenever a screen came out wrong.
4. Use `references/motion.md` for anything that moves, `ui-ux-pro-max` for concrete values and product-type patterns, `ux-designer` for flows, forms and accessibility, `web-design-guidelines` to audit existing UI code, `dataviz` for anything with numbers that deserve a picture, `apple-hig` for native Apple apps.
5. If Figma is connected, `get_design_context` on a Figma URL turns their design into buildable spec, and `use_figma` pushes a built screen back into Figma.
6. Output a **UI prompt** (recipe in `references/prompt-recipes.md`) that names the mockup, the hero element, the layout rule and the copy budget, with real values. "Make it look premium" produces nothing. "Match `docs/design/dashboard.png`. Main area fills the viewport on a 12-column grid. Hero stat spans 2 columns at 40px with a sparkline. Cards at 12px radius, 1px `#27272A` border, no shadow, 24px padding. Under 60 words outside the data" produces the thing.

Generic comes from two defaults, and the prompt names both as things to avoid: the 2024 look (purple-blue gradients, glassmorphism, emoji headings, centred hero with three cards, `shadow-lg`) and the flat clone (grey-on-grey at 13px, monospace everywhere, identical card grids, text-only stat tiles, content in a narrow centred column, no icons, nothing moving).

---

## Learn (after Sync, Unblock and Undo)

One file: **`docs/LEARNINGS.md`** in the project. Stack gotchas, decisions and why, dead ends, commands that work, anything that cost more than ten minutes to figure out. Full guidance in `references/memory.md`.

The filter: **would this change a decision next time?** If not, leave it out. A log nobody reads is worse than no log, because it makes the file too long to read.

Do this after the three modes where something was actually discovered. Skip it after Next task, Replan and Design pass, where nothing has been learnt yet - you handed over a prompt, you did not find anything out. A learn step that fires when there is nothing to learn trains you to write filler, and filler is what makes the file stop being read.

Say in one line what you added, so the user can correct it. Memory they cannot see is memory they cannot trust.

### Skill issues get a `[skill]` tag

Most learnings are about the project. Some are about this skill: a template that produced the wrong file, a prompt recipe that sent Claude Code the wrong way, a rule that was missing, a tool name that no longer exists, a host or Claude Code change the references have not caught up with. The test: **would the fix belong in build-mode rather than in this repo?** If yes, write it in `docs/LEARNINGS.md` under a `## Skill issues` heading, tagged so it can be found across projects:

```
- [skill] 2026-10-13 - prompt-recipes.md: the first-deploy recipe did not set the
  start command, so the host ran the dev server in production. Add it to Build.
```

Date, the skill file it points at, what went wrong, and the change you would make. One line each. Tell the user in the same breath: "logged a [skill] note: <one line>." Do not edit the skill itself from inside a project session; the tags get batched into one release, which is the point.

**Skill review.** When the user asks for a skill review ("any skill issues?", "what needs fixing in build mode"), search every connected project's `docs/LEARNINGS.md` for `[skill]`, group the entries by skill file, and propose one batched update. Anything that turned up in two or more projects goes first. Entries that have been fixed in a release get struck through with the release date, not deleted, so the next review does not re-raise them.

**If it would affect anyone using the skill**, not just this user's setup, offer to draft a GitHub issue for https://github.com/Samcarr01/build-mode/issues using the "Skill problem" template, with the `[skill]` line, the version from `plugin.json`, and no project code, names or secrets. The user files it; do not file it for them.

---

## Handover block

Every mode ends with this, and nothing after it. The three plain lines from [How to talk to the user](#how-to-talk-to-the-user) come first, then the part the user copies.

```
**What changed:** The sign-up page is checked and live. The next job is ready: the invite form.
**Anything risky?** No.
**What you need to do:** Open Claude Code in the project folder and type /next

---

Next job: M1-T4 - Invite form
(or paste: read docs/next-prompt.md and do it)
Come back here when it's done or if it gets stuck.
```

Then at most three bullets of anything they need to know, each technical word explained. Then stop.

---

## Reference files

Read these when the mode calls for them, not upfront.

| File | Read when |
|---|---|
| `references/tooling.md` | Kickoff capability scan, or before naming tools in a prompt. Current inventory of both sides plus the task-to-skill mapping. |
| `references/doc-pack.md` | Creating or updating any of the project docs. Has the exact templates. |
| `references/prompt-recipes.md` | Writing any prompt for Claude Code. Recipes per task type, including automations (scheduled jobs and webhooks) and the approval audit, plus the anti-patterns. |
| `references/claude-code-setup.md` | Writing `.claude/` files (including `/checkpoint`'s password scan, add-on check and reviewer stop), or the user asks about Claude Code itself, including `/goal`, mods and `/design`. Verified syntax as of Sep 2026, including the two subagent templates, with skills, rules, settings.json and permission modes re-checked Nov 2026. |
| `references/stack-picker.md` | Choosing a stack or a host, setting up Railway or any host other than Vercel, or when they ask why something costs money. |
| `references/design-brief.md` | Kickoff design steps (direction, values, screens, mockups), or a design pass. Has the full DESIGN.md template. |
| `references/design-review.md` | Sync on any task that touched a screen, and the start of every design pass. The five-check score and the diagnosis table. |
| `references/motion.md` | Kickoff design values, any UI prompt that adds motion, check 5 in Sync, and Showcase builds. Motion personality, which tool does which job, craft rules, the ban list, how to check motion, and Showcase mode. |
| `references/findability.md` | Kickoff on any public site, roadmap and prompts for public pages, findability tasks, and Sync on any task that touched a public page. SEO, GEO and AIO as one job: content in the first HTML, crawl basics, what every public page carries, receiving posts from another tool, what not to bother with, and the check. |
| `references/branch-mode.md` | Setting the live line in the roadmap step, Go live mode, and every Next task, Sync and Undo on a project whose Deploying section says branch mode. The live line and the pre-live flag rule, what a preview touches, blocking outside actions outside production, switching previews on per host, the per-task branch and pull-request lines, and who merges. |
| `references/upgrade.md` | Upgrade mode, started by the user or automatically by the setup stamp. The safe order, what may be replaced, the automatic route and its stop-and-ask cases, the setup script, and the one task added to live projects. |
| `references/memory.md` | The learn step. What is worth recording and what is filler. |

## Other skills to pull in

Do not rebuild what already exists. `references/tooling.md` has the full task-to-skill mapping and the current inventory of both sides; use it rather than a second copy here.

The ones this skill leans on most, where they are installed: `design` for direction artboards and screen mockups the user can look at before anything is coded, `ui-ux-pro-max` and `frontend-design` for design values, `dataviz` for any number that deserves a picture, `ux-designer` for flows and copy, `supabase` and `supabase-postgres-best-practices` for anything touching data, `web-design-guidelines` for auditing UI that already exists, and GreenSock's `gsap-skills` plugin for Showcase builds. None are required. Where one is missing, do the work yourself and write the values into the docs.

MCPs worth reaching for here: **Supabase**, the host's own (**Vercel**, **Railway**, or whatever `docs/TOOLING.md` names), **GitHub**, **Figma**, **claude-in-chrome** (for looking at a deployed app yourself). Name in Claude Code prompts: **Playwright**, the reason `/checkpoint` can prove a task is done rather than assert it, and **context7** for current library docs.
