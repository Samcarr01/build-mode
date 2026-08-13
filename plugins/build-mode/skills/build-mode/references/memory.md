# Memory - the self-learning part

One file, in the repo: **`docs/LEARNINGS.md`**.

That location is the whole design. Both Cowork and Claude Code can read and write it, it is version controlled so you can see when a lesson was learned, it travels with the project to another machine, and it needs no permissions from anyone. A memory file living outside the repo has to be found, and finding it means asking for access to a folder, which is a bad trade for a convenience.

## Why per-project rather than one global file

Almost everything worth remembering is specific to one codebase. "The Supabase client must be created per-request in server components" is true of this project, this framework version, this month. Filed globally it becomes noise inside a year; filed in the repo it is exactly where the next person needing it will be.

The things that genuinely are cross-project - which accounts the user has, what is installed on the Claude Code side, their design taste - are few, cheap to re-establish, and mostly belong in `docs/TOOLING.md` and `docs/DESIGN.md` per project anyway. Not worth a permission prompt at the start of every session.

Worth knowing: Claude Code also keeps its own automatic memory per repository, at `~/.claude/projects/<project>/memory/`. It is machine-local and you cannot read it, which is why `docs/LEARNINGS.md` exists in the repo instead of relying on it. The two run happily alongside each other.

## What goes in

Structure it so it can be skimmed. Headings matter more than completeness.

```markdown
# Learnings

<!-- Only things that would change a decision later. Not a diary. -->

## What works
- `npx supabase db reset` for a clean local database. Faster than fixing migration drift.
- shadcn components drop in cleanly; do not wrap them until there is a second use.

## What does not
- Auth state in React context. Fought the App Router the whole way, two sessions lost.
  Supabase's server-side session helpers instead. Do not retry the context version.

## Gotchas
- `next build` fails silently when an env var is missing at build time rather than
  runtime. Check `.env.local` first when a build breaks for no visible reason.
- Vercel does not pick up new env vars until the next deploy. Setting it is not enough.

## Decisions that got reversed
- 2026-08-07: stored the calorie estimate rather than recomputing. Recomputing meant
  last month's totals changed when the model updated.

## Commands
- Reset local DB: `npx supabase db reset`
- Regenerate types: `npx supabase gen types typescript --local > types/db.ts`
```

**What does not go in:** anything sensitive. No keys, no tokens, no customer data. This file gets committed. If a lesson involves a credential, write the shape of the problem: "the Stripe webhook secret differs between test and live mode, which broke it once" rather than the secret.

## The filter

**Would this change a decision next time?**

Yes: "Supabase Auth email templates are configured in the dashboard, not in code. Missed it twice."

No: "Built a login form on 7 August." That belongs in `docs/PROGRESS.md`, which is the log. `LEARNINGS.md` is the lesson.

The distinction matters because the two files fail in opposite directions. `PROGRESS.md` should be complete - it is history and history should not be edited. `LEARNINGS.md` should be short - it is reference, and reference stops being read when it gets long. Prune it. Three related entries become one better entry. Delete anything the codebase has since made irrelevant.

## Who writes when

**Claude Code** writes after each task, through `/checkpoint`, and when it hits a wall, through `/blocked`. Both of those skills instruct it to add to `LEARNINGS.md` only if there is something worth adding, which is why neither carries `disable-model-invocation` - a prompt that tells Claude Code to run a command it cannot run is how this loop quietly rots.

**You write** at the end of every mode, not just at the end of a project. Most of the value is in small observations from a sync or an unblock, and those are exactly the ones lost by waiting. A blocker that took an hour is the single highest-value entry there is - it is the one most likely to recur and the one most expensive when it does.

Say in one line what you added: "noted that the env var needs setting in Vercel too, since that caught you out again." They can then correct it, and memory they cannot see is memory they cannot trust.

## When it is working

You will know because the user stops being asked the same question. If you find yourself asking which database they are using, or what the deploy command is, or whether the repo is private, the answer was supposed to be in `CLAUDE.md`, `ARCHITECTURE.md` or `TOOLING.md`. Re-asking is the clearest signal the memory is not being read - fix where the answer lives, not just the answer.
