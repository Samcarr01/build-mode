# Prompt recipes

The prompt is the product. Everything else in this skill exists to make the prompt good. A weak one costs the user an hour of Claude Code confidently building the wrong thing, and they will not spot it until they open the app.

## The shape

Every prompt has the same six parts, in this order. The order matters: context before instruction, instruction before constraint, and the done-check last so it is the thing still in view when Claude Code decides whether to stop.

```markdown
# <M1-T3> <Task name>

## Context
<Two or three sentences. What exists now, what this adds, why it matters to a user.>

**Read first:** CLAUDE.md, docs/ARCHITECTURE.md (<section>), docs/DESIGN.md (<section>)
**Use these:** <skills and MCP tools for this task, from docs/TOOLING.md>

## Build
1. <Step, concrete enough to be unambiguous>
2. <Step>
3. <Step>

## Constraints
- <Things not to do, and things that must hold>

## Done when
- [ ] <Verifiable by clicking something or running something>
- [ ] <...>
- [ ] `npm run build` completes and there are no new type errors

## Before you finish
- Run `@agent-code-reviewer` on the files you changed. Fix anything it marks critical
  or warning, say what you changed, then run `/checkpoint`.
- Run `/checkpoint`. It builds, clicks through what you made, then commits. It pushes
  only if this project's Deploying rules say that is safe, and asks first otherwise.
- Do not start the next task.
```

The reviewer line is left out of exactly two prompts: the scaffold task, which is generated boilerplate, and a trivial one-file change, the same case that skips plan mode. Everything else gets it. The reviewer cannot edit, so "fix anything it marks" is an instruction to the main session, and the prompt names the files because a subagent cannot see the conversation.

And a plan-mode line at the top of every prompt except a trivial one-file change:

```
Start in plan mode. Show me the plan and wait for approval before editing anything.
In the plan, list in plain English anything that deletes data, changes the database,
sends or posts anything, costs money to run, or cannot be undone. If there is none,
say "Nothing risky".
```

Default this on rather than off. The user cannot read a diff, so the plan is the only review the work gets. A minute spent reading a plan is the cheapest possible place to catch a wrong direction.

The second half matters as much as the first. A plan is mostly file names and steps, and a migration or a new send call sits in that list looking like everything else. Asking for the risky steps in plain words puts the one part the user can judge at the top, and "Nothing risky" is a line they can check in two seconds.

## Writing each part

**Context.** Claude Code has no memory of the last session and no idea what the product is for. Two sentences of why turns a mechanical implementation into a sensible one. Say what a user is trying to do, not what the code should look like.

**Read first.** Name files and sections, not "the docs". Pointing at `docs/ARCHITECTURE.md (§Auth)` gets that section read. Saying "check the docs" gets nothing read.

**Use these.** One line naming the skills and MCP tools that apply. Claude Code will not hunt for a skill it has not been reminded of, and will happily hand-roll a colour palette that `ui-ux-pro-max` would have got right. See `tooling.md` for the mapping.

**Build.** Numbered steps, each one thing. Concrete enough that two different people would build the same thing. Not so prescriptive that you are writing the code in English - if you find yourself specifying variable names, you have gone too far and should let Claude Code do its job.

**Flag-before-build.** One line, and it does more than any constraint: it invites Claude Code to say "step 4 conflicts with the day-boundary rule in ARCHITECTURE.md" before writing code rather than after. Prompts are entirely imperative by default, and an imperative prompt gets compliance even where the instruction was wrong.

**Constraints.** Where you spend the leverage. "Do not add a new dependency for this", "keep it in one file for now", "do not touch the auth flow", "no new colours outside DESIGN.md". Constraints prevent scope creep better than any amount of instruction.

**Done when.** The most important section. Every tick must be checkable by clicking something or running something. Not "auth works properly" but "signing up with an email that already exists shows an error rather than a blank page". If you cannot write it as something observable, the task is not defined well enough yet - go back and split it.

Four ticks or fewer, plus the build tick. More than that and the task is too big for one session.

Write them so **Claude Code can check them itself in a browser**, not so the user has to. "Signing up with an existing email shows an error rather than a blank page" is something a browser can be driven through. "The flow feels right" is not. `/checkpoint` walks these ticks in a real browser before it is allowed to write `[x]`, so a tick phrased as an observable click is worth three phrased as a judgement.

One thing to check before you finalise them: does any tick depend on something a later task builds? "User A cannot read user B's rows" needs two accounts, which may not exist yet. If so, rewrite it as something checkable today, or move it to the task where it becomes checkable.

## Recipes by task type

### Scaffold - the first task in a project

```markdown
# M0-T1 Project skeleton

## Context
New project: <one line on what it is>. The planning docs are already in the folder,
nothing else is. This task creates the app and pushes it, so the deploy task that
follows can prove the pipeline before there is anything complicated to debug.

## Build
1. **First, get the folder out of the scaffolder's way.** `create-next-app` refuses to
   run in a directory containing files it does not recognise, and `CLAUDE.md` is one of
   them. So: `mv CLAUDE.md docs/CLAUDE.md.tmp` before you start, and move it back
   afterwards. (`docs/` and `.claude/` are both ignored by the scaffolder, so they can
   stay where they are.)
2. Create a <framework> app with TypeScript, Tailwind and the App Router, in this
   directory.
3. Move `CLAUDE.md` back to the project root. Check it is there before continuing.
4. Set up <UI library> with the base config.
5. Replace the default homepage with a single page showing the project name.
6. Add a `.gitignore` covering `.env*`, `node_modules`, `.next`. Add a `.env.example`
   listing the variable names with no values.
7. Initialise git, commit, push to a new **private** GitHub repo called `<name>`.
8. Stop there. Do not build any features.

## Constraints
- No extra dependencies beyond the above.
- Private repo. <One line on why: this holds real data / it is not ready to be public.>
- Do not scaffold folders for features that do not exist yet. Empty directories
  become a filing system nobody agreed to.
- The `docs/` files and the `.claude/` folder already exist and are not yours to
  change. Do not overwrite, move or tidy them.

## Done when
- [ ] `npm run dev` serves a page at localhost:3000 showing the project name
- [ ] `npm run build` completes without errors
- [ ] `CLAUDE.md` is back in the project root and `.claude/skills/next/SKILL.md` still exists
- [ ] The repo exists on GitHub with one commit

## Before you finish
- Run `/checkpoint`.
- Do not start the next task.
```

Step 1 is not fussiness. It is a verified failure: with `CLAUDE.md` in the folder,
`create-next-app` prints *"The directory contains files that could conflict"* and exits.
Without that line, the very first prompt of every project stalls, and Claude Code
improvises around a collision the plan created.

### First deploy - getting it live on the chosen host

The task that proves the pipeline, usually `M0-T3`. Write it for the host in the Hosting section of `docs/TOOLING.md`. The `[@]` task before it (account created, project made, repo connected) must be ticked first, because Claude Code cannot create accounts or click through a host's sign-up.

This is the one prompt that pre-approves a push, because nothing is live yet and there is no real data. Every later task goes back to whatever the Deploying section of `CLAUDE.md` says.

```markdown
# M0-T3 First deploy to <host>

## Context
The app exists and is on GitHub (M0-T1). The user has created the <host> project and
connected the repo (M0-T2). This task gets the empty app live and makes it say which
commit it is running, so every later check can confirm the right code is deployed.

## Build
1. Add a `/version` route that returns the deployed commit SHA from
   `<the host's commit variable, from docs/TOOLING.md>`, or "local" when it is unset.
   Plain text, no styling, no auth. If the variable only exists at build time
   (Netlify, Cloudflare), bake it in during the build.
2. Add only the config this host needs to build and start the app.
   <Vercel: none. Railway: usually none, but the app must listen on the `PORT` Railway
   provides. Fly.io: `fly.toml`, from `fly config save -a <app>` if the user already
   created the app, or `fly launch --no-deploy` if not. Anything else: what the host's
   docs say for this framework.>
3. List every environment variable the app needs in `.env.example`, names only, and
   tell me which ones I must set in <host> before the deploy works. Do not set them.
4. Read the Deploying section of `CLAUDE.md` and check it against the host's docs:
   what a push does, and whether anything runs against real data on deploy. Correct it
   if it is wrong, and say what you changed.

## Constraints
- No features. No database unless the roadmap puts it in this task.
- Never commit a secret, a `.env` file or a host token.
- Do not change settings in the host's dashboard. If something there needs changing,
  tell me what and where.

## Done when
- [ ] `/version` on localhost returns "local"
- [ ] `.env.example` lists every variable the app reads, with no values
- [ ] The Deploying section of `CLAUDE.md` matches what this host actually does
- [ ] `npm run build` completes without errors

## Before you finish
- Run `@agent-code-reviewer` on the files you changed. Fix anything it marks critical
  or warning, say what you changed, then run `/checkpoint`.
- Run `/checkpoint`. The push is pre-approved for this task only; Claude Code will still
  show its push prompt, because of the `ask` rule in settings.json, and I will approve it.
- <Host deploys on push:> Once pushed, watch the deploy with <the host's MCP or CLI,
  from docs/TOOLING.md> until it finishes, then open `/version` on the live URL.
  <Host deploys by command:> Once pushed, give me the exact deploy command and wait.
  When I say it is deployed, open `/version` on the live URL.
- If the live SHA matches the commit `/checkpoint` pushed, add the live URL to that
  PROGRESS entry and commit the one-line change without pushing it. If the deploy
  failed or the SHA is different, change the task to `[!]` and say what you saw.
- Do not start the next task.
```

The `/version` route looks like a nicety and is not. It is the one check Sync can run against any host, with or without an MCP: open the URL, read the SHA, compare it with `PROGRESS.md`. Without it, "the deploy is green" and "the deploy is running the code we just wrote" are two different claims that look the same.

### Feature

The standard shape above. The thing to get right is a Context that says what the user is doing, and a Done-when that a non-developer could check.

### UI or screen

```markdown
# M2-T1 Dashboard layout

## Context
Users land here after logging in. Right now they see a blank page. This is the frame
the rest of the app hangs off, so getting the structure right matters more than the
detail inside it.

**Read first:** docs/DESIGN.md in full (§Layout, §Hierarchy, §Copy, §Screens > Dashboard),
docs/ARCHITECTURE.md (§Routes)
**Match this:** `docs/design/dashboard.png`. Open it before you write a line. The picture
wins on layout and hierarchy; the token table wins on exact values.
**Use these:** the `ui-ux-pro-max` skill for the stat tile and sidebar patterns,
`ux-designer` for the empty state and nav labels.

## The screen in one line
A stranger should be able to say in five seconds that this is where they see how
their projects are doing, and that "New project" is the thing to do next.

## Build
1. Sidebar nav, 240px, items: <list> with Lucide icons at 20px. Active state per DESIGN.md.
2. Main area fills the rest of the viewport on a 12-column grid, 32px padding.
   No max width on the page. Page header: title left, primary action button top right.
3. Stat row: the hero stat (<which>) spans 4 columns at display size with a 7-day
   sparkline; the three secondary stats span 2 columns each at h1 size.
   "<ratio stat>" is a progress ring, not a fraction in text.
4. Recent list below: status pill with colour, title, one-line meta. Rows lift 1px on hover.
5. Empty state: "<heading, 6 words max>", "<body, 12 words max>", one primary button.
6. Responsive: at 768 the sidebar collapses to a top bar; at 390 the stat row stacks
   with the hero first.

## Constraints
- Only the tokens in docs/DESIGN.md. No new colours, no new font sizes.
- Copy budget: labels 3 words, no helper text on this screen, under 60 words outside
  the data. Anything explaining how the product works goes in a tooltip.
- Not the 2024 look: no gradients, glassmorphism, emoji headings, `shadow-lg`.
- Not the flat clone: no grey-on-grey at 13px, no monospace outside IDs, no grid of
  identical cards, no text-only stat tiles, no empty side margins, nothing static.
- Real content, not lorem ipsum. Write the actual words.
- Do not wire up data. Static layout with realistic sample values, this task.

## Done when
- [ ] Screenshot at 1440 matches `docs/design/dashboard.png` in layout and hierarchy
- [ ] The hero stat is visibly the largest element and has its sparkline
- [ ] It is usable at 390px wide with no horizontal scrolling, hero stat first
- [ ] Keyboard tab order goes through nav then main, with visible focus rings
```

Three things make this recipe work, and all three are missing from a token-only prompt. **The picture**: Claude Code reads images, and a mockup settles layout and hierarchy in a way no list of tokens can. **The one-line purpose**: it tells Claude Code what the screen is for, so it can make a hundred small choices in that direction. **Both anti-pattern lists**: the 2024 defaults and the flat clone are each strong pulls in every model, and banning only the first lands you in the second. A general instruction to "look premium" overrides neither.

### Data model or migration

```markdown
# M1-T1 Database schema for <feature>

## Context
<What data this stores and what reads it.>

**Read first:** docs/ARCHITECTURE.md (§Data model)
**Use these:** the `supabase` and `supabase-postgres-best-practices` skills. Use the
Supabase MCP to inspect the current schema before writing anything - do not guess at
what is already there.

## Build
1. <Table>: columns, types, constraints as in ARCHITECTURE.md.
2. Row level security so a user reads and writes only their own rows.
3. A migration file, not a change made by hand in the dashboard.
4. Regenerate TypeScript types.

## Constraints
- Migrations only. Anything done by hand in the dashboard is lost on the next environment.
- Every table gets RLS. A table without it is readable by anyone with the public key.
- Do not drop or rename an existing column without flagging it first.
- Run `@agent-code-reviewer` on the migration and the code that calls it before you
  finish, even if the change is small. This touches user data, and a second model
  reading it is the only review it will get.

## Done when
- [ ] The migration applies cleanly to a fresh database
- [ ] `get_advisors` reports no new security warnings
- [ ] A signed-in user can read their own rows and not another user's
- [ ] Types are regenerated and the build passes
```

Schema is the most expensive thing to change later, so it gets the most careful prompt. Always inspect before writing.

**On Postgres without Supabase** (Railway Postgres, or any other host's), the shape is the same and four lines change. **Use these:** `supabase-postgres-best-practices` for the schema, `context7` for the current Drizzle docs, and the host's MCP to confirm the database service and its `DATABASE_URL`. Step 2 becomes "every query for this table goes through the scoping helper named in `CLAUDE.md`", because there is no RLS. Step 3 becomes a Drizzle migration generated with `drizzle-kit`, run by the pre-deploy command, never `drizzle-kit push` against production. And the `get_advisors` tick becomes "every new query for this table goes through the scoping helper; `@agent-code-reviewer` finds none that skip it".

### Integration with a third-party service

```markdown
# M3-T2 <Service> integration

## Context
<What it does for the user and where it appears.>

**Use these:** <MCP if there is one>. Use the `context7` MCP to pull the current docs
for <library> rather than working from memory - this library changes often and a
half-remembered API costs an hour.

## Build
1. <Setup, including which env vars are needed>
2. <The happy path>
3. <Error handling: what the user sees when the service is down or rejects the request>

## Constraints
- Keys in environment variables. Never in code, never committed.
- Handle the failure case. A third-party service will be down at some point and
  the app should say something useful rather than white-screen.
- Do not build a retry loop without a cap.

## Done when
- [ ] The happy path works end to end
- [ ] With a deliberately wrong key, the user sees a clear message and nothing crashes
- [ ] No key appears in the repo: `git grep -i "<key prefix>"` returns nothing
```

### AI feature

The recipe most likely to be needed and least like the others, because the output is not deterministic. Three decisions have to be made in the prompt or they get made badly by default.

```markdown
# M2-T2 Calorie estimate from a food description

## Context
The user types what they ate in plain words. We send that to a model and get back a
calorie number and a confidence. This is the feature the app exists for.

**Read first:** docs/ARCHITECTURE.md (§AI), docs/DESIGN.md (§States)

## Build
1. A server-side route that takes the text and returns structured output:
   `{ calories: number, confidence: "high" | "medium" | "low", items: [...] }`.
   Use the provider's structured output mode, not a "reply in JSON" instruction.
2. Validate the response against the schema before using it. If it does not parse,
   retry once, then return a clear error. Never show unvalidated model output as a number.
3. The instructions the model gets live in `prompts/calorie-estimate.md`, not in a
   string inside the route. Sections: the job in one line, always, never, what to
   return when the input is not food, and one worked example of an input and its
   output. A `PROMPT_VERSION` line at the top.
4. **Store the result on the entry**, do not recompute on read, with the model name and
   prompt version beside it. <Or the reverse, with the reason.>
5. Show the confidence in the UI: <how>. A low-confidence estimate must not look like
   a measured fact.
6. Loading state while it thinks. Error state when it fails.
7. `prompts/calorie-estimate.samples.json`: eight inputs, two ordinary, two awkward,
   two that are not food at all, two empty or nonsense, each with the shape its answer
   must have. A test runs them against the real model with a low token cap and checks
   the shape and confidence, not the exact wording.

## Constraints
- API key server-side only. It must never reach the browser.
- Cap the cost: <model choice> and a max token limit. One call per submission,
  no automatic retries beyond the single reparse.
- Do not let a failed estimate block saving the entry. The log is the point;
  the number is an enhancement.
- If anything the model writes goes to someone outside the app, decide whether a
  person approves it first. See the Automation recipe.

## Done when
- [ ] Typing "two eggs and toast" returns a number in under 5 seconds
- [ ] The samples test passes, and the not-food and nonsense samples come back as low
      confidence or a clear "could not estimate", not a confident wrong number
- [ ] With the API key removed, the entry still saves and the UI explains why
      there is no estimate
- [ ] The stored number does not change when the page is reloaded
```

The four decisions to force in every AI prompt:

**Store or recompute.** The most consequential and the easiest to get wrong. If you recompute on read, every historical number silently changes when the model updates, and the user's totals from last month are no longer what they saw. Store the result and the model version. Recompute only when the user asks.

**What happens when it fails or returns rubbish.** Models return malformed output sometimes. Say what the user sees. "It crashes" is the default if you do not.

**Cost.** Name the model, cap the tokens, one call per action. Tell the user roughly what a thousand uses costs and suggest a spend limit on the provider dashboard on day one. This is the only line in a project that can run away while they are asleep.

**Where the app's own prompt lives, and how it is tested.** A prompt written as a string inside a route gets edited in passing by an unrelated task, and nobody notices the answers changed. In its own file, with a version and a samples test, changing it is a deliberate act that shows up in the plan, and the stored version says which prompt produced last month's numbers. The samples are also the cheapest way to check a model upgrade: run them before switching, not after someone notices. If the build is for a client, write the samples from their real inputs, with names and amounts changed, because the awkward ones are what the tool will actually meet.

And note the done-checks: none of them says "returns the right answer", because it cannot. They check the shape, the failure path, the honesty of the UI and the stability of the stored value. That is what a verifiable done-check looks like for non-deterministic output.

### Automation: scheduled jobs, webhooks, anything that acts on its own

Use this for any task where the app does something nobody clicked (a schedule, a webhook from another system, a queue) or reaches outside itself (sends, posts, pays, deletes, writes into someone else's software). Judge by what the task does, not by what the app is: a booking tool, a CRM add-on and a personal habit tracker can each have one.

Nobody is watching when one of these goes wrong. A bug on a screen shows one person a wrong page. A bug here sends forty customers the same message twice at three in the morning. Four decisions go in the prompt, or they get made badly by default.

The example below is one illustration. Swap in the project's own job, systems and timings; keep the shape.

```markdown
# M2-T3 Draft the overdue invoice reminders

## Context
Every weekday at 9am London time the app finds invoices more than 7 days overdue and
drafts a reminder for each. A person approves them in the review screen before anything
is sent. This task builds the job that finds and drafts. It does not send.

**Read first:** CLAUDE.md (§Standing rules), docs/ARCHITECTURE.md (§Jobs and outbound
actions, §Data model)
**Use these:** <the host's MCP for the job's deploys and logs>, `context7` for the current
<provider> API docs.

## Build
1. One function, `draftOverdueReminders(runDate)`, that does the whole job and can be run
   by hand, by the schedule, or by a test. The schedule only ever calls it.
2. Every item it acts on gets a run key: <invoice id + reminder number>. Before acting,
   look the key up and skip the item if it is there.
3. A `job_runs` table: one row per run with started, finished, found, drafted, skipped
   and any error. Every run writes one, including runs that find nothing.
4. Drafts go into the approval queue with status `waiting`. Nothing in this task calls
   the provider's send endpoint.
5. The schedule, in local time. <If the host's scheduler only runs in UTC:> schedule it
   at both UTC hours that can be 9am in London (`0 8,9 * * 1-5`); the function checks the
   London clock, exits on the run that is not 9am locally, and the run key stops a double
   run. <If the scheduler takes a time zone:> set `Europe/London` there instead. Put the
   reason in a comment by the schedule. If the schedule is a setting in the host's
   dashboard, end your reply by telling me exactly what to paste there. The process
   exits when the job ends.
6. `JOBS_ENABLED` environment variable. Unless it is `true`, the function writes a
   skipped row to `job_runs` and does nothing else.

## Constraints
- <If outbound actions need approval:> The only code that sends, posts, pays or deletes
  anything outside the app is `<the approved-action helper named in CLAUDE.md>`, and it
  refuses any item not marked approved by a named person. Do not call the provider from
  anywhere else.
- Respect <provider>'s rate limit of <n per minute>: back off on a 429, at most 3 retries.
- After downtime, run once for today. Never once per missed day.
- `JOBS_ENABLED` stays unset locally and in preview environments.
- No scheduling library when the host's own scheduler does the job.

## Done when
- [ ] Running the function twice for the same date drafts each reminder once, not twice
- [ ] A run with nothing overdue still writes a `job_runs` row saying 0 found
- [ ] With `JOBS_ENABLED` unset, a run writes "skipped" and drafts nothing
- [ ] <If approval applies:> A search of the repo finds the provider's send call only
      inside the approved-action helper
```

The four decisions to force in every automation prompt:

**Run it twice and nothing changes.** Schedulers fire twice, a deploy restarts a job halfway, and a retry after a timeout repeats work that half succeeded. Providers resend webhooks the same way. A key per item, checked before acting, turns all of those into nothing happening instead of a duplicate email. For a webhook coming in, the key is the provider's event id, and the signature is checked before anything else.

**Decide who presses send.** The first time a roadmap task reaches outside the app, ask the user once whether a person should approve each item before it goes, and recommend yes for anything that lands in someone else's inbox, feed or bank account. Write the answer in CLAUDE.md under Standing rules and in docs/ARCHITECTURE.md §Jobs and outbound actions. If the answer is yes, enforce it in code: one helper that refuses unapproved items, and no other path to the provider. A line in CLAUDE.md is a request; the helper holds even when a later task forgets the rule. The code reviewer checks it on every task, and the approval audit proves it at the end of the phase.

**A record of every run.** When someone asks "did it go out on Tuesday?", the run table answers in ten seconds. Without it the answer is a log search that may have rotated away. It is also the proof a job ran: the newest row says what happened, where a green deploy only says the code started. `/checkpoint` reads it after a manual run, and a Done-when tick can name it.

**An off switch and a clock.** `JOBS_ENABLED` stops a misbehaving job without a deploy, which is the difference at 3am. And name the time zone in the code. Railway cron and Vercel's older Cron Jobs both run in UTC, so a "9am" job moves by an hour at every clock change unless the code handles it. On Vercel, use Schedules rather than `crons` in `vercel.json`: a schedule takes a `timezone`, and on Hobby it can run as often as every 15 minutes rather than once a day, though each run can start up to 15 minutes after the time set. A schedule that calls a `path` is a public URL, so the route checks `CRON_SECRET` before doing anything. On any other host, find out before writing the task which time zone its scheduler uses and what it does with a run that is still going (Railway skips the next one).

**The `[@]` task before it.** If the schedule has to be created in the host's dashboard (a Railway cron service does), that is the user's step. Give it its own `[@]` task directly before the job's build task, and say exactly where the setting is.

### Approval audit

The last task of any phase that added a way to send, post, pay or delete, in a project where a person approves outbound actions. It earns its own task because the failure it looks for does not show up when everything works: it shows up the first time a retry, an admin route or a forgotten script acts without anyone approving it.

```markdown
# M2-T7 Approval audit

## Context
This phase added <the reminder job and the send step>. This project's rule (CLAUDE.md,
Standing rules) is that nothing sends, posts, pays or deletes outside the app without a
named person approving it. This task proves the code makes that impossible, not just
unlikely.

**Read first:** CLAUDE.md (§Standing rules), docs/ARCHITECTURE.md (§Jobs and outbound actions)
**Use these:** `@agent-code-reviewer` on every file that reaches outside the app, once
the list in step 1 exists.

## Build
1. List every place the code reaches the outside world: provider SDK calls, outbound
   HTTP, email, outgoing webhooks, file deletes, payments. Search the whole repo, not
   only this phase's files.
2. For each one, trace whether it can run without passing through the approved-action
   helper. Include scheduled jobs, retries, admin routes, seed scripts and anything a
   webhook coming in can trigger.
3. Try to get round it, each as a test: call the send route directly with an item that
   is not approved, replay an item that was already sent, approve as a user from a
   different tenant.
4. Fix anything that gets through, one finding at a time.

## Constraints
- No new features. Findings and fixes only.
- If a path is meant to act without approval because the user decided it should, stop
  and say which one. Do not decide it is fine.

## Done when
- [ ] Every outbound path is in docs/ARCHITECTURE.md §Jobs and outbound actions, with what gates it
- [ ] Each attempt in step 3 has a test, and every one is refused
- [ ] Nothing found is left unfixed, or it is marked `[!]` on the roadmap with the reason
```

If the project has nothing that reaches outside the app, the task is ticked with "nothing outbound" and the ARCHITECTURE section says "none".

### Debugging

Different shape. The goal is a good diagnosis, not a fast fix.

~~~markdown
# Fix: <symptom in plain words>

## What happens
<Exactly what the user sees or does. Steps to reproduce.>

## Error
```
<the exact error text, unedited>
```

## What we know
- Started after <M1-T3 / a dependency update / nothing obvious>
- <Anything already ruled out>
- docs/LEARNINGS.md <mentions something similar / has nothing on this>

## Where to look first
1. <Most likely cause and why>
2. <Second>
3. <Third>

**Use these:** `@agent-test-debug-runner` first, to reproduce the failure with the
narrowest command and report the cause without editing anything. Then
`superpowers:systematic-debugging` if installed - root cause before fixes.
<Plus the database and host tools from docs/TOOLING.md: Supabase MCP query_logs,
Vercel MCP get_runtime_errors, Railway MCP get-logs, or the host's CLI logs command.>

## How to approach it
Find the cause before changing anything. Say what you think is wrong and why,
then fix that one thing. If the first fix does not work, do not stack a second on
top of it - undo it and reconsider, because two half-fixes are much harder to
reason about than one clean failure.

## Done when
- [ ] <The original symptom is gone, described as something to click>
- [ ] <Nothing else broke: name the thing most likely to have been affected>
- [ ] The cause is written into docs/LEARNINGS.md, not just the fix
~~~

Do not put your suspected answer in the Build section unless you are genuinely confident. A confident wrong diagnosis is worse than none - Claude Code will pursue it well past the point where it should have stopped.

### Refactor or cleanup

```markdown
# M4-T1 <what is being tidied>

## Context
<What is messy and what it is costing.>

## Build
1. <The change>

## Constraints
- Behaviour must not change. This is a tidy-up, not a redesign.
- One concern at a time. Do not fix unrelated things you notice - note them instead.
- Small commits.

## Done when
- [ ] Everything that worked before still works: <name the specific flows to check>
- [ ] The build passes and no new type errors
- [ ] Anything noticed but not fixed is listed in PROGRESS.md
```

## Anti-patterns

Things that reliably produce bad sessions.

**Multiple tasks in one prompt.** "Build auth and the dashboard and wire up billing" gets you three half-finished things and a context window with no room left. One task.

**Splitting one task across subagents.** "Use a subagent for the API and another for the UI" sounds efficient and is not. The steps of one task depend on each other and touch the same files, which are the two cases Anthropic says never to parallelise, and the user cannot review what three workers did. Subagents appear in exactly two places: the reviewer before `/checkpoint`, and the test runner at the start of a debugging prompt. If a task really has independent halves, it is two tasks on the roadmap.

**Vague quality words.** "Make it clean and modern", "production-ready", "polished". These mean nothing operationally, so the model falls back on its defaults, which is exactly the generic look you were trying to avoid. Replace with values and constraints.

**No done-check.** Without it, Claude Code decides for itself when it is finished, and it is optimistic. Every prompt gets one.

**Writing the code in English.** If the prompt specifies function names and file structure, you have done the thinking Claude Code is better at. Say what and why; let it decide how.

**Assuming context.** "Continue where we left off" means nothing to a fresh session. Every prompt stands alone.

**Silence on the docs.** Without an explicit instruction to update PROGRESS.md and ROADMAP.md, they rot within three sessions and the whole system stops working. Every prompt ends with `/checkpoint`.

**Done-checks only a human can run.** "It looks right", "the spacing feels balanced", "the flow makes sense". The user is the slowest and least reliable test runner in this system, and they only run each check once. If a tick can be rewritten as something a browser tool can click, rewrite it. Save their attention for the two or three judgement calls that genuinely need eyes.
