# Picking the stack

The user is not a developer, so this is your call, not theirs. Make it, state it in one line with the reason, and move on. A menu of options handed to someone with no basis to choose between them is not helpfulness, it is passing the buck.

Two things to get right, because both are expensive to undo: **where the data lives** and **where it runs**. Everything else can change later without much pain.

**Check current pricing before you quote it.** Free tiers and limits move around. Web-search the actual pricing page at kickoff rather than repeating a number from memory, and give the user a range with the date you checked. If the search fails or the page will not fetch, say "roughly £X, worth checking before you commit" and link the pricing page rather than dropping the number - a caveated estimate is useful, a confident wrong one is not, and silence leaves them with no idea what this costs.

## The default

Unless something below says otherwise:

- **Next.js** with TypeScript, App Router
- **Tailwind** with **shadcn/ui** components
- **Supabase** for database, auth and file storage
- **Vercel** for hosting
- **GitHub** for the repo

Why this and not something else: it is the combination with the most examples in the world, which matters because Claude Code writes better code where there is more prior art. Auth, database and storage arrive as one decision rather than three. And the free tiers cover a project until it has real users.

## When to move off the default

### Railway instead of Vercel

Reach for Railway when the thing needs to **keep running** rather than respond and stop:

- A background worker, a queue, a scheduled job that takes minutes
- WebSockets or anything holding a long-lived connection
- A Python or Go service alongside the frontend
- A long-running AI job - Vercel's serverless functions have a time limit that a slow model call will hit

Railway runs a container that stays up, which suits all of that. It starts costing money sooner than Vercel's hobby tier, so it is a reasonable call when the shape fits but not the default. The usual arrangement is frontend on Vercel, worker on Railway, both talking to the same Supabase.

If the app is request-response - user clicks, server responds, done - Vercel is simpler and cheaper. Do not reach for Railway out of a vague sense that it is more "real".

### Something other than Supabase

Rarely. Supabase is Postgres, which is the right default database for almost everything. Move off it only when:

- The data is genuinely a document store with no relationships, and even then Postgres `jsonb` usually wins
- There is an existing database to connect to
- Real-time collaborative editing is the core feature, which is a specialist problem

### Not a web app at all

- **iOS or macOS native**: SwiftUI. Pull in the `apple-hig` skill. Note that the user will need a Mac, Xcode and a £79/year Apple Developer account to put it on a phone, and App Store review takes days - say this at kickoff, not at the end.
- **Cross-platform mobile**: Expo with React Native. Reuses what they know from React, and Expo handles most of the build and distribution pain.
- **A script or automation**: Python, run on a schedule. Do not build a web app around something that could be a cron job.
- **An internal tool for one person**: consider whether it needs auth at all. A single-user tool behind a password is a day of work saved.

## Cost, in plain English

The user should know what starts costing money and when, before it does.

- **Nothing until there are users.** Every service above has a free tier that covers development and early use.
- **The first thing to cost money** is usually the database, when it exceeds the free storage or row limits, or a free-tier project gets paused for inactivity.
- **Then hosting**, when traffic grows or a commercial project needs a paid plan.
- **A domain** is the one guaranteed cost. Roughly £10 to £15 a year for a `.com`.
- **AI API calls cost per use** and are the one line that can surprise. If the project calls a model, say roughly what a thousand uses costs and suggest a spending cap on day one.

Give the number as a range, with what triggers the jump. "Free until roughly a few hundred users, then about £20 to £25 a month" is more useful than a precise figure that will be wrong.

## Things to decide at kickoff, not later

These are cheap now and painful in three weeks.

| Decision | Why now |
|---|---|
| Auth: which sign-in methods | Adding a provider later means a migration of existing users |
| Multi-user or single-user | Changes every database table. The most expensive thing to retrofit. |
| Does it take payments | Changes the data model and adds a compliance surface |
| File uploads | Storage, size limits and access rules are structural |
| Does it need to work offline | Changes the whole architecture. Almost always no. |

Ask about these in the interview if the answer is not obvious from what the user described.

## Things not to decide at kickoff

Do not spend the interview on these. Pick the obvious thing and move.

- State management. Start with what the framework gives you.
- Testing framework. Add when there is something worth testing.
- Analytics, monitoring, error tracking. Week two problems.
- Component libraries beyond the default.
- CI beyond what the host does automatically.

Every one of these is a decision the user has no basis to make and no reason to care about. Deciding them yourself and not mentioning it is the right move.

## Writing it up

In `docs/ARCHITECTURE.md`, a table with a one-line reason per row. In chat, three lines maximum:

> Next.js on Vercel, Supabase for the database and login. That combination has the
> most examples for Claude Code to work from, and it is free until you have real
> users. The one cost now is a domain, about £12 a year.

Then stop. They do not need the alternatives you considered unless they ask.
