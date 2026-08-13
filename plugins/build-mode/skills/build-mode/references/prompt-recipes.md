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

## Before you finish
- Run `/checkpoint`.
- Do not start the next task.
```

And a plan-mode line at the top of every prompt except a trivial one-file change:

```
Start in plan mode. Show me the plan and wait for approval before editing anything.
```

Default this on rather than off. If the user cannot read a diff, the plan is the only review the work gets. A minute spent reading a plan is the cheapest possible place to catch a wrong direction.

## Writing each part

**Context.** Claude Code has no memory of the last session and no idea what the product is for. Two sentences of why turns a mechanical implementation into a sensible one. Say what a user is trying to do, not what the code should look like.

**Read first.** Name files and sections, not "the docs". Pointing at `docs/ARCHITECTURE.md (§Auth)` gets that section read. Saying "check the docs" gets nothing read.

**Use these.** One line naming the skills and MCP tools that apply. Claude Code will not hunt for a skill it has not been reminded of, and will happily hand-roll a colour palette that `ui-ux-pro-max` would have got right. See `tooling.md` for the mapping.

**Build.** Numbered steps, each one thing. Concrete enough that two different people would build the same thing. Not so prescriptive that you are writing the code in English - if you find yourself specifying variable names, you have gone too far and should let Claude Code do its job.

**Flag-before-build.** One line, and it does more than any constraint: it invites Claude Code to say "step 4 conflicts with the day-boundary rule in ARCHITECTURE.md" before writing code rather than after. Prompts are entirely imperative by default, and an imperative prompt gets compliance even where the instruction was wrong.

**Constraints.** Where you spend the leverage. "Do not add a new dependency for this", "keep it in one file for now", "do not touch the auth flow", "no new colours outside DESIGN.md". Constraints prevent scope creep better than any amount of instruction.

**Done when.** The most important section. Every tick must be checkable by clicking something or running something. Not "auth works properly" but "signing up with an email that already exists shows an error rather than a blank page". If you cannot write it as something observable, the task is not defined well enough yet - go back and split it.

Four ticks or fewer. More than that and the task is too big for one session.

Two things to check before you finalise them. Does any tick depend on a tool Claude Code may not have, like an MCP server? Does any tick depend on something a later task builds - "user A cannot read user B's rows" needs two accounts, which may not exist yet? If so, rewrite it as something checkable today, or move it to the task where it becomes checkable.

## Recipes by task type

### Scaffold - the first task in a project

```markdown
# M0-T1 Project skeleton

## Context
New project: <one line on what it is>. Nothing exists yet. This task creates the
repo and gets an empty page deployed, so the deploy pipeline is proven before there
is anything complicated to debug.

## Build
1. Create a <framework> app with TypeScript, Tailwind and the App Router.
2. Set up <UI library> with the base config.
3. Replace the default homepage with a single page showing the project name.
4. Add a `.gitignore` covering `.env*`, `node_modules`, `.next`. Add a `.env.example`
   listing the variable names with no values.
5. Initialise git, commit, push to a new **private** GitHub repo called `<name>`.
6. Stop there. Do not build any features.

## Constraints
- No extra dependencies beyond the above.
- Private repo. <One line on why: this holds real data / it is not ready to be public.>
- Do not scaffold folders for features that do not exist yet. Empty directories
  become a filing system nobody agreed to.
- The `docs/` files already exist. Do not overwrite them.

## Done when
- [ ] `npm run dev` serves a page at localhost:3000 showing the project name
- [ ] `npm run build` completes without errors
- [ ] The repo exists on GitHub with one commit

## Before you finish
- Run `/checkpoint`.
- Do not start the next task.
```

### Feature

The standard shape above. The thing to get right is a Context that says what the user is doing, and a Done-when that a non-developer could check.

### UI or screen

```markdown
# M2-T1 Dashboard layout

## Context
Users land here after logging in. Right now they see a blank page. This is the frame
the rest of the app hangs off, so getting the structure right matters more than the
detail inside it.

**Read first:** docs/DESIGN.md in full, docs/ARCHITECTURE.md (§Routes)
**Use these:** the `ui-ux-pro-max` skill for spacing and type scale, `ux-designer`
for the empty state and nav labels.

## Build
1. Sidebar nav, 240px, items: <list>. Active state per docs/DESIGN.md.
2. Main area, max width 1200px, 32px padding.
3. Empty state: <heading>, <one line of body>, one primary button.
4. Responsive: sidebar collapses to a top bar below 768px.

## Constraints
- Only the tokens in docs/DESIGN.md. No new colours, no new font sizes.
- No gradients. No glassmorphism. No emoji in headings. No `shadow-lg`.
- Real content, not lorem ipsum. Write the actual words.
- Do not wire up data. Static layout only, this task.

## Done when
- [ ] The dashboard renders at `/dashboard` and matches docs/DESIGN.md
- [ ] Nav highlights the current page
- [ ] It is usable at 375px wide with no horizontal scrolling
- [ ] Keyboard tab order goes through nav then main, with visible focus rings
```

Naming the anti-patterns explicitly is what keeps it from looking AI-generated. They are strong defaults in every model, and a general instruction to "look premium" will not override them.

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

## Done when
- [ ] The migration applies cleanly to a fresh database
- [ ] `get_advisors` reports no new security warnings
- [ ] A signed-in user can read their own rows and not another user's
- [ ] Types are regenerated and the build passes
```

Schema is the most expensive thing to change later, so it gets the most careful prompt. Always inspect before writing.

### Integration with a third-party service

```markdown
# M3-T2 <Service> integration

## Context
<What it does for the user and where it appears.>

**Use these:** <MCP if there is one>. Check the official docs for the current API
rather than working from memory - this library changes often.

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
3. **Store the result on the entry**, do not recompute on read. <Or the reverse, with
   the reason.>
4. Show the confidence in the UI: <how>. A low-confidence estimate must not look like
   a measured fact.
5. Loading state while it thinks. Error state when it fails.

## Constraints
- API key server-side only. It must never reach the browser.
- Cap the cost: <model choice> and a max token limit. One call per submission,
  no automatic retries beyond the single reparse.
- Do not let a failed estimate block saving the entry. The log is the point;
  the number is an enhancement.

## Done when
- [ ] Typing "two eggs and toast" returns a number in under 5 seconds
- [ ] Typing nonsense returns low confidence or a clear "could not estimate",
      not a confident wrong number
- [ ] With the API key removed, the entry still saves and the UI explains why
      there is no estimate
- [ ] The stored number does not change when the page is reloaded
```

The three decisions to force in every AI prompt:

**Store or recompute.** The most consequential and the easiest to get wrong. If you recompute on read, every historical number silently changes when the model updates, and the user's totals from last month are no longer what they saw. Store the result and the model version. Recompute only when the user asks.

**What happens when it fails or returns rubbish.** Models return malformed output sometimes. Say what the user sees. "It crashes" is the default if you do not.

**Cost.** Name the model, cap the tokens, one call per action. Tell the user roughly what a thousand uses costs and suggest a spend limit on the provider dashboard on day one. This is the only line in a project that can run away while they are asleep.

And note the done-checks: none of them says "returns the right answer", because it cannot. They check the shape, the failure path, the honesty of the UI and the stability of the stored value. That is what a verifiable done-check looks like for non-deterministic output.

### Debugging

Different shape. The goal is a good diagnosis, not a fast fix.

```markdown
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

**Use these:** <Supabase MCP get_logs / Vercel MCP get_runtime_errors / etc.>

## How to approach it
Find the cause before changing anything. Say what you think is wrong and why,
then fix that one thing. If the first fix does not work, do not stack a second on
top of it - undo it and reconsider, because two half-fixes are much harder to
reason about than one clean failure.

## Done when
- [ ] <The original symptom is gone, described as something to click>
- [ ] <Nothing else broke: name the thing most likely to have been affected>
- [ ] The cause is written into docs/LEARNINGS.md, not just the fix
```

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

**Vague quality words.** "Make it clean and modern", "production-ready", "polished". These mean nothing operationally, so the model falls back on its defaults, which is exactly the generic look you were trying to avoid. Replace with values and constraints.

**No done-check.** Without it, Claude Code decides for itself when it is finished, and it is optimistic. Every prompt gets one.

**Writing the code in English.** If the prompt specifies function names and file structure, you have done the thinking Claude Code is better at. Say what and why; let it decide how.

**Assuming context.** "Continue where we left off" means nothing to a fresh session. Every prompt stands alone.

**Silence on the docs.** Without an explicit instruction to update PROGRESS.md and ROADMAP.md, they rot within three sessions and the whole system stops working. Every prompt ends with `/checkpoint`.
