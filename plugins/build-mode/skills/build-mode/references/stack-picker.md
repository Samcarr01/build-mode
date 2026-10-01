# Picking the stack

The user is probably not a developer, so this is your call, not theirs. Make it, state it in one line with the reason, and move on. A menu of options handed to someone with no basis to choose between them is not helpfulness, it is passing the buck.

Two things to get right, because both are expensive to undo: **where the data lives** and **where it runs**. Everything else can change later without much pain.

**Check current pricing before you quote it.** Free tiers and limits move around. Web-search the actual pricing page at kickoff rather than repeating a number from memory, and give the user a range with the date you checked. If the search fails or the page will not fetch, say "roughly £X, worth checking before you commit" and link the pricing page rather than dropping the number - a caveated estimate is useful, a confident wrong one is not, and silence leaves them with no idea what this costs.

## The default

Unless something below says otherwise:

- **Next.js** with TypeScript, App Router
- **Tailwind** with **shadcn/ui** components
- **Supabase** for database, auth and file storage
- **Hosting: whatever the user asked for.** If they said "you decide", **Vercel** for a request-response app and **Railway** for anything that has to keep running. See [Hosting](#hosting).
- **GitHub** for the repo

Why this and not something else: it is the combination with the most examples in the world, which matters because Claude Code writes better code where there is more prior art. Auth, database and storage arrive as one decision rather than three. And on Vercel the free tiers cover a project until it has real users; Railway charges from the first month.

## Hosting

Hosting is the one stack decision that is the user's to make if they want to. Everything else in this file is your call.

**If they named a host in the interview, that is the host.** They may already pay for it, know its dashboard, or have a client who insists on it. Do not re-open it to win an argument about which host is best; a host they know beats a better one they have to learn. Push back only when it **cannot run the shape of the thing**, and then make the smallest change that fixes it. A static host cannot run a background worker, so keep their host for the front end and put the one service that does not fit somewhere that does. Say why in one line.

**If they said "you decide"**, pick by what the app does:

| The app | Host | Why |
|---|---|---|
| Request-response: user clicks, server answers, done | **Vercel** | Simplest, preview deploy per branch, free hobby tier. Hobby is personal, non-commercial use only, so anything that earns money (including paid client work) needs Pro. |
| Has to **keep running**: a background worker, a queue, a job that takes minutes, WebSockets, a Python or Go service, a slow AI call | **Railway** | A container that stays up, with no serverless time limit for a slow model call to hit. Front end, API, worker and database can all live in one project. |
| Static site, no server at all | Whatever they already have, otherwise Vercel | Every host does this well. Do not overthink it. |
| Mostly request-response, plus one worker | **Vercel** for the front end, **Railway** for the worker, one Supabase for both | The usual split. Each half runs where it fits. |

Do not reach for Railway out of a vague sense that it is more "real", and do not keep a long-running job on Vercel because it was the default.

Whichever host it is, write it in three places at kickoff, because three different readers need it: the Hosting row of `docs/ARCHITECTURE.md` (why), the **Hosting** section of `docs/TOOLING.md` (the live URL, how deploys and logs work, read by Sync and Unblock), and the **Deploying** section of `CLAUDE.md` (what a push does, read by `/checkpoint`). Templates for all three are in `doc-pack.md` and `tooling.md`.

### Vercel

The path with the fewest moving parts. Connect the GitHub repo and every push to `main` deploys to production, every other branch gets its own preview URL. Environment variables live in the Vercel dashboard and need a redeploy to take effect. The deployed commit is in `VERCEL_GIT_COMMIT_SHA`. Cowork reads deploys through the Vercel MCP, and Claude Code can have it too through `.mcp.json`.

### Railway

A full path, not just the place the worker goes. Use it whenever the user asks for it, or when the table above points there.

**How a Railway app is laid out.** One Railway **project** per app. Inside it, one **service** per thing that deploys separately: `web`, `api`, `worker`, and the database if it lives on Railway too. Services talk to each other over Railway's private network. Never put two unrelated apps in one project; a project is the unit of billing, variables and access.

**What a push does.** Once a service is connected to the GitHub repo, Railway deploys it on every push to the connected branch, `main` by default. Migrations, where Railway runs them, go in the service's **pre-deploy command** (`deploy.preDeployCommand` in `railway.json`), which executes between build and start against the real database. So on Railway a push to `main` is a production deploy, and with Railway Postgres it may change the schema too. The Deploying section of `CLAUDE.md` must say so in exactly those words, so that `/checkpoint` asks before pushing. If the repo has GitHub Actions, Railway's **Wait for CI** setting holds the deploy until they pass; switch it on when tests exist.

**Variables.** Set in Railway, never committed. Wire services together with reference variables (`${{Postgres.DATABASE_URL}}`), which resolve over the private network, rather than pasting a public connection string. Variable changes are staged in Railway and only take effect when deployed, which catches people out once; when it does, it goes in `LEARNINGS.md`. The deployed commit is in `RAILWAY_GIT_COMMIT_SHA` when the deploy came from GitHub.

**Cost.** Railway has no free tier that will run a live app. New accounts get a one-time trial credit, and the free plan after it carries a small monthly credit that will not keep a service up all month, so plan on a paid plan plus usage. Say so at kickoff, check the current plan prices on the pricing page, and quote the range. A worker that runs all month is the line that grows.

**Tools.** Cowork: the Railway connector (deploys, build and runtime logs, error rate, metrics, variables). Claude Code: the Railway plugin, `/plugin install railway@claude-plugins-official`, which brings Railway's hosted MCP server and its `use-railway` skill, or `railway setup agent` if they already use the Railway CLI. Record which side has what in `docs/TOOLING.md`.

**Where the data lives on Railway.** Two good answers. Pick one at kickoff and write it into `docs/ARCHITECTURE.md`, because moving data later is the expensive kind of change.

- **Supabase alongside Railway** (the default). The app runs on Railway and talks to Supabase over the internet. Auth, storage, RLS and backups stay Supabase's job, and the free tier still covers early use. Choose this unless one of the reasons below applies.
- **Railway Postgres in the same project.** One bill, one dashboard, the database on the private network next to the app. Choose it when the user asks for it, when the project is commercial and paying for Supabase Pro on top of Railway would be a second bill for one app, or when the data must not leave the project. The cost of that choice is that four things Supabase gave for free become the app's job:

| Supabase gave you | With Railway Postgres it is | Rule |
|---|---|---|
| Schema and migrations | **Drizzle** with `drizzle-kit` | Migrations are files in the repo and run in the pre-deploy command. Nothing is changed by hand in Railway's database view. Prisma only if the project already uses it. |
| Auth | **Better Auth** with the Drizzle adapter | Sessions in Postgres. Never roll sign-in by hand. |
| Row level security | The API layer | There is no RLS by default. Every query is scoped by the signed-in user (or their organisation) in one helper or middleware, and that rule goes in the Standing rules of `CLAUDE.md` so it loads every session. It is the line the `code-reviewer` subagent checks. A query that skips the helper is a data leak, and nothing else will catch it. |
| File storage | A **Railway bucket** (S3-compatible, private) | Uploads go through the API with signed URLs. Never a public bucket. |
| Backups | Railway volume backups | Switch on scheduled backups for the Postgres volume in Phase 0, as an `[@]` task, before there is real data. A restore is a reviewed change, never a casual click. |

Say the trade out loud at kickoff: auth, storage and backups become three roadmap tasks rather than three toggles in a dashboard, roughly a session each. And do not let the project quietly pick up `@supabase/*` because it is what the examples use.

### Any other host

Netlify, Cloudflare, Render, Fly.io, a VPS, a company server, whatever the user already runs. The loop works on any of them, because it never needed Vercel; it needs five answers about the host. Find them at kickoff, before the roadmap, by searching the host's own docs (try "`<host>` Claude Code", "`<host>` MCP" and "`<host>` environment variables"), then write them into the **Hosting** section of `docs/TOOLING.md`:

1. **What triggers a deploy?** A push to a branch, a CLI command, a GitHub Action, a button in a dashboard.
2. **Does a deploy touch real data?** Migrations, seed scripts, anything that runs against production on the way up.
3. **Where are deploy status and build logs?** An MCP server (check `ListConnectors` here and the host's docs for Claude Code), a CLI on the user's machine, or only a dashboard.
4. **Where are runtime logs and errors?** Same three options.
5. **Which variable carries the deployed commit SHA?** So the app can show which commit is live. If the host has none, Claude Code passes the SHA in at build time.

Answers 1 and 2 go into the Deploying section of `CLAUDE.md` as well. If the host has no MCP on either side, say so in the **Not available** list of `docs/TOOLING.md`: Sync then relies on the live-commit check and the browser, and Unblock asks Claude Code to run the host's CLI for logs, or asks the user to paste them.

Starting points, checked October 2026. Confirm against the host's docs at kickoff; these change.

| Host | A deploy happens on | Commit SHA variable | Watch for |
|---|---|---|---|
| Vercel | Push to the connected branch; preview per branch | `VERCEL_GIT_COMMIT_SHA` | Hobby is non-commercial only |
| Railway | Push to the connected branch | `RAILWAY_GIT_COMMIT_SHA` | Pre-deploy command runs migrations against production |
| Netlify | Push to the connected branch; preview per pull request | `COMMIT_REF` (build time only) | Bake the SHA in at build; functions do not see build variables by default |
| Render | Push to the connected branch (auto-deploy) | `RENDER_GIT_COMMIT` | Free services sleep when idle, so the first visit is slow |
| Cloudflare | Push to the connected branch, with Workers Builds or Pages | `WORKERS_CI_COMMIT_SHA` on Workers Builds, `CF_PAGES_COMMIT_SHA` on Pages (both build time) | Next.js runs on Workers through the OpenNext adapter, not on Pages. Workers is not full Node; check the libraries |
| Fly.io | `fly deploy`, or a GitHub Action you add | None built in; pass it as a build argument | A push alone deploys nothing |
| Your own server | Whatever you set up | Whatever you pass in | TLS, restarts, backups and updates are all yours |

The same five questions work for things that are not websites. An Expo app's deploy is an EAS build, a native app's is a TestFlight upload, a script's is the scheduled job. Answer them anyway; Sync needs to know where to look.

## When to move off the default

### Something other than Postgres

Rarely. Supabase is Postgres, and so is Railway Postgres, and Postgres is the right default database for almost everything. Move off it only when:

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

- **Nothing until there are users**, on the default stack. Vercel and Supabase both have free tiers that cover development and early use.
- **Except on Railway**, and container hosts in general: a service that stays up costs money from the first month after the trial credit runs out. Small, but not zero, so say it at kickoff rather than when the first bill lands.
- **The first thing to cost money** is usually the database, when it exceeds the free storage or row limits, or a free-tier project gets paused for inactivity.
- **Then hosting**, when traffic grows or a commercial project needs a paid plan. On Vercel that is the moment the project starts earning money, because Hobby is non-commercial only.
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

Or, when the user asked for Railway:

> Next.js on Railway, as you asked, with Supabase for the database and login. Railway
> keeps the app running with no time limit on slow jobs; Supabase's free tier covers
> the data. Railway charges from the first month, roughly £<n> at this size (checked
> <date>), plus about £12 a year for a domain.

Then stop. They do not need the alternatives you considered unless they ask.
