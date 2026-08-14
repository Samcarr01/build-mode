# The doc pack - templates

Six documents. Between them they hold everything the project knows, so that neither Claude Code nor Cowork has to remember anything between sessions. Fill the placeholders with real values; a template left half-generic is worse than no template, because it reads as if a decision was made when it was not.

Each file has one job. Keep them to it:

| File | Question it answers | Who writes it |
|---|---|---|
| `CLAUDE.md` | How do I work on this project? | Cowork, rarely changed |
| `docs/ARCHITECTURE.md` | How is it built and why? | Cowork, changed on replan |
| `docs/DESIGN.md` | What does it look like? | Cowork, changed on design pass |
| `docs/ROADMAP.md` | What is left and in what order? | Both |
| `docs/PROGRESS.md` | What happened? | Claude Code mostly, Cowork adds verification |
| `docs/LEARNINGS.md` | What do we now know that we did not? | Both |

---

## CLAUDE.md

Loaded into every Claude Code session, so every line costs tokens forever. Under 200 lines, no exceptions. Point at the other docs rather than importing them - an `@import` loads at launch and would drag the whole roadmap in every time.

```markdown
# <Project name>

<One sentence: what it does and who for.>

## Stack
- <Framework and version>
- <Database / backend>
- <Hosting>
- <Anything else load-bearing>

## Commands
| What | Command |
|---|---|
| Run locally | `npm run dev` |
| Build | `npm run build` |
| Lint | `npm run lint` |
| Typecheck | `npx tsc --noEmit` |
| Deploy | <or "push to main, Vercel deploys automatically"> |

<!-- Add a Test row only once tests actually exist. A command listed here that does
     not run is worse than no row: /checkpoint will try it and report a false failure. -->

## Read before you start a task
- `docs/ROADMAP.md` - what is being built and in what order
- `docs/PROGRESS.md` - what happened last session
- `docs/ARCHITECTURE.md` - the data model and the decisions already made
- `docs/DESIGN.md` - before building any screen
- `docs/LEARNINGS.md` - the mistakes already made once
- `docs/TOOLING.md` - which skills and MCP servers to use

## How we work
- One task per session. The task is in `docs/next-prompt.md`. Do that and stop.
- Use plan mode by default. Show the plan and wait before editing. Skip it only for
  a one-file change that is obviously safe.
- When the task is done, run `/checkpoint` before doing anything else. It builds,
  typechecks, drives the new flow in a browser with the Playwright MCP, then commits
  and pushes. A task is not done until that has passed.
- If you are stuck after two real attempts, run `/blocked` instead of guessing again.
  A clear description of the problem is worth more than a third attempt.

## Standing rules
- <Project-specific conventions Claude Code cannot infer from the code.>
- Never commit secrets. Environment variables go in `.env.local`, which is gitignored.
- Ask before adding a dependency that does something the stack already does.
- <Naming conventions, folder layout, anything that differs from the framework default.>

## Superpowers
<!-- Include this section ONLY if the user has the superpowers plugin. Leave it out
     entirely otherwise: instructions about an absent plugin are noise. -->
The superpowers plugin is installed and loads every session. For this project:

- **Do not run `brainstorming`.** The design work is already done. Requirements are in
  `docs/ROADMAP.md`, decisions in `docs/ARCHITECTURE.md`, visual rules in `docs/DESIGN.md`.
  Read those instead of asking. If a task looks wrong against them, say so before you
  build - that is the review, not a reason to start a fresh design conversation.
- **Do not run `writing-plans`.** The plan is `docs/ROADMAP.md` and the current task is
  `docs/next-prompt.md`. Do not write plans into `docs/superpowers/`.
- **Do use `verification-before-completion`** on every task. Evidence before claims.
- **Do use `systematic-debugging`** for any bug. Root cause before fixes.
- **Do use `requesting-code-review`** before anything touching auth, payments or user data.
- Skip `using-git-worktrees` and `subagent-driven-development`. One task, one branch here.

## The person you are working with
The user is probably not a developer. They may ship fast and understand systems, but they will not
catch a bad call by reading a diff. So:
- Explain what you did in plain English, not in diff summaries.
- Flag anything irreversible - schema changes, deletions, deploys - before doing it.
- If a decision has a real trade-off, say so and recommend one. Do not present a menu.
```

Two sections do the heavy lifting there. **Superpowers** stops a plugin that reloads on
every `/clear` from re-opening decisions that were already made, while keeping the three
of its skills that genuinely protect someone who cannot read a diff. **The person you
are working with** changes how Claude Code reports back for the whole project.


---

## docs/ARCHITECTURE.md

```markdown
# Architecture

## Stack and why
| Layer | Choice | Why |
|---|---|---|
| Frontend | <e.g. Next.js 15, App Router> | <one line> |
| Styling | <e.g. Tailwind + shadcn/ui> | <one line> |
| Database | <e.g. Supabase Postgres> | <one line> |
| Auth | <e.g. Supabase Auth, email + Google> | <one line> |
| Hosting | <e.g. Vercel> | <one line> |
| Payments | <e.g. Stripe, or "none yet"> | <one line> |

Running cost: about £<n> a month at launch. <What starts costing money and when.>

## Data model
<Every table: columns, types, relationships. Plain text or SQL, but complete -
this is the thing most expensive to change later, so it is worth being explicit.>

### <table_name>
| Column | Type | Notes |
|---|---|---|
| id | uuid | primary key |
| ... | | |

Access rules: <who can read and write what. For Supabase, the RLS policy in words.>

## Routes and screens
| Path | What it is | Auth |
|---|---|---|
| `/` | Landing | public |
| ... | | |

## Environment variables
| Name | What it is | Where it comes from |
|---|---|---|
| `NEXT_PUBLIC_SUPABASE_URL` | Supabase project URL | Supabase dashboard, Settings > API |
| ... | | |

## Decisions
Newest first. Record anything you would otherwise be asked to re-justify.

**<date> - <the decision>**
Why: <reason>. Alternative considered: <what and why not>.
Reversible: <yes/no, and what it would cost to change>.

## Open questions
- <Thing not yet decided, when it needs deciding by, and what it blocks.>
```

---

## docs/DESIGN.md

Built in the design step. Full guidance in `design-brief.md`. The rule that matters: **real values only**. "Modern and clean" tells Claude Code nothing. `#0A0A0A` tells it exactly what to do.

---

## docs/ROADMAP.md

```markdown
# Roadmap

**Now:** M1-T3 - <task name>
**Next:** M1-T4 - <task name>
**Shipped:** <one line on what currently works>

Status key: [ ] not started, [~] in progress, [?] built but not verified,
[x] done and verified, [!] blocked, [@] waiting on the user, ~~cut~~

---

## Phase 0 - Skeleton
Goal: something deployed and reachable on the internet.

- [x] `M0-T1` Repo, Next.js app, pushed to GitHub
- [@] `M0-T2` **Needs you:** create the Vercel account and connect the GitHub repo
- [ ] `M0-T3` Deployed to Vercel, custom domain pointing at it
- [ ] `M0-T4` Supabase project connected, one test read working

## Phase 1 - <the smallest useful version>
Goal: <what a user can do at the end of this phase>.

- [ ] `M1-T1` <task>
      Done when: <the single most important check>
- [ ] `M1-T2` <task>

## Phase 2 - <next>
...

## Later
Not scheduled. Here so they stop taking up space in conversation.
- <idea>

## Cut
- ~~`M1-T5` <task>~~ - <why it went>
```

Rules that keep this useful:

- **IDs are permanent.** `M1-T3` means the same task forever. Never renumber. PROGRESS entries and old prompts point at these.
- **`[?]` is the honest default** when there is no evidence yet. `/checkpoint` writes `[x]` only after it has built the project and walked the flow in a browser, and `[!]` when either fails. If you are looking at a `[x]` with no build result and no click recorded in PROGRESS.md, treat it as `[?]` and check.
- **`[@]` is for the steps only the user can do**: creating an account, pasting an API key into Vercel, pointing a domain, approving something in a dashboard. Give these their own task IDs rather than burying them inside a build task, because a build task that secretly needs the user is the most common way a project stalls for a week. Say exactly what they have to do and where.
- **The three lines at the top** are what the user reads on their phone. Keep them current.
- Tasks are one session each. If the Definition of Done needs more than four ticks, split it.

---

## docs/PROGRESS.md

Append at the top. Never edit or delete history - a wrong entry that was later corrected is itself useful information.

```markdown
# Progress

<!-- Newest at the top. Claude Code writes after each task; Cowork adds verification notes. -->

## 2026-08-07 - M1-T3 - Sign-up form
**By:** Claude Code
**Commit:** `a4f9c21`
**Built:** Email and password sign-up at `/signup`, wired to Supabase Auth.
Validation on both sides, error states for taken email and weak password.
**Files:** `app/signup/page.tsx`, `components/auth-form.tsx`, `lib/supabase/client.ts`
**Checked:** Build passes, no type errors. Drove it in the browser: signed up with a
new email and landed on the redirect, then tried the same email again and got the
"already registered" error rather than a blank page.
**Not done:** No email verification yet - that is M1-T5.
**Unsure about:** Redirect after sign-up goes to `/dashboard`, which does not exist yet.

> **Cowork check 2026-08-07:** Verified against the deployed preview. Form and
> validation are there. Redirect target confirmed missing, added as `M1-T4`. Marked `[x]`.
```

Two lines carry most of the weight. **Commit** is the project's undo button - without a SHA on every entry, "put it back to before M1-T3" is archaeology rather than a command. **Checked** is what separates a real tick from an optimistic one: it records what was actually clicked and seen, so a later session can tell the difference between verified and assumed.

When you decided something on the user's behalf because you could not ask - which this skill tells you to do constantly - record it here too, so it is visible and cheap to reverse:

```markdown
## Assumptions - 2026-08-07
Decided without asking, flag anything wrong and it gets changed:
- Days roll over at 04:00 rather than midnight, so a late meal counts to the right day.
- Two users maximum, no invite flow. Adding one later is a morning's work.
```

The **Unsure about** line is the highest-value line in the file. It is where the next session's problems get flagged before they cost anything.

---

## docs/LEARNINGS.md

```markdown
# Learnings

<!-- Only things that would change a decision later. Not a diary. -->

## Stack and setup
- Supabase client must be created per-request in server components. A module-level
  client leaks auth state between users. Cost 40 minutes on M1-T2.

## Gotchas
- `next build` fails silently if an env var is missing at build time rather than
  runtime. Check `.env.local` first when a build breaks for no visible reason.

## Decisions that did not work
- Tried putting auth state in React context. Fought the App Router the whole way.
  Using Supabase's server-side session helpers instead. Do not retry the context version.

## Commands that work
- Reset the local database: `npx supabase db reset`
```

The filter: **would this change a decision next time?** If not, leave it out.
